<template>
    <div class="search ncm-page">
        <mu-tabs class="search-tab"
            inverse
            :value="tab"
            @change="handleTabChange">
            <mu-tab value="song">单曲</mu-tab>
            <mu-tab value="artist">歌手</mu-tab>
            <mu-tab value="album">专辑</mu-tab>
            <mu-tab value="playlist">歌单</mu-tab>
            <mu-tab value="video">视频</mu-tab>
            <mu-tab value="user">用户</mu-tab>
        </mu-tabs>
        <div class="search-content">
            <CenteredLoading v-if="ui.search.pending"></CenteredLoading>
            <CenteredTip v-else-if="ui.search.keyword === ''"
                icon="search"
                tip="右上角搜索框内输入，回车搜索！"></CenteredTip>
            <CenteredTip v-else-if="ui.search.result.items && ui.search.result.items.length === 0"
                icon="inbox"
                tip="哎呀～什么都没找到 ..."></CenteredTip>
            <CenteredTip v-else-if="ui.search.error"
                icon="error_outline"
                :tip="`出错了 ... ${ui.search.error.code}：${ui.search.error.msg}`"></CenteredTip>
            <TrackList v-else-if="tab === 'song'"
                :tracks="ui.search.result.items"
                :indexOffset="searchOffset"></TrackList>
            <UserList v-else-if="tab === 'user'"
                :users="ui.search.result.items"></UserList>
            <ArtistList v-else-if="tab === 'artist'"
                :list="ui.search.result.items"></ArtistList>
            <AlbumList v-else-if="tab === 'album'"
                showArtist
                :list="ui.search.result.items"></AlbumList>
            <PlaylistList v-else-if="tab === 'playlist'"
                :list="ui.search.result.items"></PlaylistList>
            <VideoList v-else-if="tab === 'video'"
                showBadge
                :videos="ui.search.result.items"></VideoList>
            <CenteredTip v-else
                icon="bug_report"
                tip="为什么会这样呢 ..."></CenteredTip>
            <div class="pagination"
                v-if="paginationShow">
                <mu-pagination :total="ui.search.result.total"
                    :current="currentPage"
                    :page-size="pageSize"
                    @change="handlePageChange">
                </mu-pagination>
            </div>
        </div>
    </div>
</template>

<script>
import { mapActions } from 'vuex';

import TrackList from '@/components/TrackList/TrackList.vue';
import CenteredTip from '@/components/CenteredTip.vue';
import CenteredLoading from '@/components/CenteredLoading.vue';
import UserList from '@/components/UserList/UserList.vue';
import VideoList from '@/components/VideoList.vue';
import AlbumList from '@/components/AlbumList.vue';
import ArtistList from '@/components/ArtistList.vue';
import PlaylistList from '@/components/PlaylistList.vue';

export default {
    data() {
        return {
            tab: 'song',
            pageSize: 20,
            currentPage: 1
        };
    },
    computed: {
        /** @returns {import('@/store/modules/ui').State} */
        ui() { return this.$store.state.ui; },
        /** @returns {number} */
        searchOffset() {
            return (this.currentPage - 1) * this.pageSize;
        },
        /** @returns {boolean} */
        paginationShow() {
            const { result: { total } } = this.ui.search;
            return total > this.pageSize;
        }
    },
    methods: {
        ...mapActions(['search']),
        updateQueryString() {
            this.$router.push({
                name: 'search',
                query: {
                    keyword: this.$route.query.keyword,
                    type: this.tab
                }
            });
        },
        handleTabChange(val) {
            this.tab = val;
            this.currentPage = 1;
            this.updateQueryString();
        },
        handlePageChange(val) {
            this.currentPage = val;
            this.handleSearch();
        },
        async handleSearch() {
            let { keyword, type } = this.$route.query;
            if (!keyword) return;
            if (!type) type = this.tab; else this.tab = type;
            if (keyword === this.ui.search.keyword &&
                type === this.ui.search.type &&
                this.searchOffset === this.ui.search.offset) return;
            if (keyword !== this.ui.search.keyword) this.currentPage = 1;
            await this.search({ keyword, type, limit: this.pageSize, offset: this.searchOffset });
        }
    },
    created() {
        const { type, offset } = this.ui.search;
        if (type) {
            this.tab = type;
        }
        this.currentPage = offset / this.pageSize + 1;
    },
    beforeRouteEnter(to, from, next) {
        // this component would be created in the new route
        next(vm => {
            vm.handleSearch();
        });
    },
    beforeRouteUpdate(to, from, next) {
        // this component is reused in the new route
        next();
        this.handleSearch();
    },
    components: {
        TrackList,
        CenteredTip,
        CenteredLoading,
        UserList,
        VideoList,
        AlbumList,
        ArtistList,
        PlaylistList
    }
};
</script>

<style lang="less">
.search {
    height: 100%;
    display: flex;
    flex-direction: column;
    .search-tab {
        z-index: 10;
        box-shadow: 0 0 12px rgba(0, 0, 0, 0.4);
    }
    .search-content {
        height: calc(~'100% - 48px');
        overflow: auto;
    }
    .centered-loading {
        height: calc(~'100% - 78px');
    }
    .pagination {
        padding: 16px 16px 30px;
    }
}
</style>
