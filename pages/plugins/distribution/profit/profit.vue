<template>
    <view :class="theme_view">
        <!-- 导航 -->
        <view class="nav-base bg-white">
            <block v-for="(item, index) in nav_status_list" :key="index">
                <view :class="'item fl tc cr-grey ' + (nav_status_index == index ? 'cr-main nav-active-line' : '')" :data-index="index" @tap="nav_event">{{ item.name }}</view>
            </block>
        </view>

        <!-- 列表 -->
        <scroll-view :scroll-y="true" class="scroll-box scroll-box-ece-nav" @scrolltolower="scroll_lower" lower-threshold="60">
            <view v-if="data_list.length > 0" class="data-list padding-horizontal-main padding-top-main">
                <view v-for="(item, index) in data_list" :key="index" class="item padding-main border-radius-main oh bg-white spacing-mb">
                    <view class="base oh br-b padding-bottom-main">
                        <text class="cr-base">{{ item.add_time }}</text>
                        <text class="fr cr-main">{{ item.status_name }}</text>
                    </view>
                    <view class="content margin-top">
                        <navigator :url="'/pages/plugins/distribution/profit-detail/profit-detail?id=' + item.id" hover-class="none">
                            <block v-for="(fv, fi) in content_list" :key="fi">
                                <view class="single-text margin-top-xs">
                                    <text class="cr-grey margin-right-xl">{{ fv.name }}</text>
                                    <text class="cr-base">{{ item[fv.field] }}</text>
                                    <text v-if="(fv.unit || null) != null" class="cr-grey">{{ fv.unit }}</text>
                                </view>
                            </block>
                        </navigator>
                    </view>
                </view>
            </view>
            <view v-else>
                <!-- 提示信息 -->
                <component-no-data :propStatus="data_list_loding_status"></component-no-data>
            </view>

            <!-- 结尾 -->
            <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>
        </scroll-view>
    </view>
</template>
<script>
const app = getApp();
import componentNoData from "../../../../components/no-data/no-data";
import componentBottomLine from "../../../../components/bottom-line/bottom-line";

export default {
    data() {
        return {
            theme_view: app.globalData.get_theme_value_view(),
            data_list: [],
            data_total: 0,
            data_page_total: 0,
            data_page: 1,
            data_list_loding_status: 1,
            data_bottom_line_status: false,
            data_is_loading: 0,
            params: null,
            nav_status_list: [
                { name: this.$t('common.all'), value: "-1" },
                { name: this.$t('profit.profit.3c7zmg'), value: "0" },
                { name: this.$t('profit.profit.67o785'), value: "1" },
                { name: this.$t('profit.profit.l5knxu'), value: "2" },
                { name: this.$t('detail.detail.32171c'), value: "3" },
            ],
            nav_status_index: 0,
            content_list: [
                { name: this.$t('order-detail.order-detail.x3ge6c'), field: "total_price" },
                { name: this.$t('order-detail.order-detail.v52n5r'), field: "refund_price" },
                { name: this.$t('profit.profit.utg512'), field: "profit_price" },
                { name: this.$t('profit.profit.6a7t71'), field: "level_name" },
            ],
        };
    },

    components: {
        componentNoData,
        componentBottomLine,
    },
    props: {},

    onLoad(params) {
        // 调用公共事件方法
        app.globalData.page_event_onload_handle(params);

        // 是否指定状态
        var nav_status_index = 0;
        if ((params.status || null) != null) {
            for (var i in this.nav_status_list) {
                if (this.nav_status_list[i]["value"] == params.status) {
                    nav_status_index = i;
                    break;
                }
            }
        }
        this.setData({
            params: params,
            nav_status_index: nav_status_index,
        });
        this.init();
    },

    onShow() {
        // 调用公共事件方法
        app.globalData.page_event_onshow_handle();

        // 分享菜单处理
        app.globalData.page_share_handle();
    },

    // 下拉刷新
    onPullDownRefresh() {
        this.setData({
            data_page: 1,
        });
        this.get_data_list(1);
    },

    methods: {
        init() {
            var user = app.globalData.get_user_info(this, "init");
            if (user != false) {
                // 用户未绑定手机则转到登录页面
                if (app.globalData.user_is_need_login(user)) {
                    uni.redirectTo({
                        url: "/pages/login/login?event_callback=init",
                    });
                    return false;
                } else {
                    // 获取数据
                    this.get_data_list();
                }
            } else {
                this.setData({
                    data_list_loding_status: 0,
                    data_bottom_line_status: false,
                });
            }
        },

        // 获取数据
        get_data_list(is_mandatory) {
            // 分页是否还有数据
            if ((is_mandatory || 0) == 0) {
                if (this.data_bottom_line_status == true) {
                    uni.stopPullDownRefresh();
                    return false;
                }
            }

            // 是否加载中
            if (this.data_is_loading == 1) {
                return false;
            }
            this.setData({
                data_is_loading: 1,
                data_list_loding_status: 1,
            });

            // 加载loding
            uni.showLoading({
                title: this.$t('common.loading_in_text'),
            });

            // 获取数据
            var status = (this.nav_status_list[this.nav_status_index] || null) == null ? -1 : this.nav_status_list[this.nav_status_index]["value"];
            uni.request({
                url: app.globalData.get_request_url("index", "profit", "distribution"),
                method: "POST",
                data: {
                    page: this.data_page,
                    status: status,
                    is_more: 1,
                },
                dataType: "json",
                success: (res) => {
                    uni.hideLoading();
                    uni.stopPullDownRefresh();
                    if (res.data.code == 0) {
                        if (res.data.data.data.length > 0) {
                            if (this.data_page <= 1) {
                                var temp_data_list = res.data.data.data;
                            } else {
                                var temp_data_list = this.data_list || [];
                                var temp_data = res.data.data.data;
                                for (var i in temp_data) {
                                    temp_data_list.push(temp_data[i]);
                                }
                            }
                            this.setData({
                                data_list: temp_data_list,
                                data_total: res.data.data.total,
                                data_page_total: res.data.data.page_total,
                                data_list_loding_status: 3,
                                data_page: this.data_page + 1,
                                data_is_loading: 0,
                            });

                            // 是否还有数据
                            this.setData({
                                data_bottom_line_status: this.data_page > 1 && this.data_page > this.data_page_total,
                            });
                        } else {
                            this.setData({
                                data_list_loding_status: 0,
                                data_list: [],
                                data_bottom_line_status: false,
                                data_is_loading: 0,
                            });
                        }
                    } else {
                        this.setData({
                            data_list_loding_status: 0,
                            data_is_loading: 0,
                        });
                        if (app.globalData.is_login_check(res.data, this, "get_data_list")) {
                            app.globalData.showToast(res.data.msg);
                        }
                    }
                },
                fail: () => {
                    uni.hideLoading();
                    uni.stopPullDownRefresh();
                    this.setData({
                        data_list_loding_status: 2,
                        data_is_loading: 0,
                    });
                    app.globalData.showToast(this.$t('common.internet_error_tips'));
                },
            });
        },

        // 滚动加载
        scroll_lower(e) {
            this.get_data_list();
        },

        // 导航事件
        nav_event(e) {
            this.setData({
                nav_status_index: e.currentTarget.dataset.index || 0,
                data_page: 1,
            });
            this.get_data_list(1);
        },
    },
};
</script>
<style>
@import "./profit.css";
</style>
