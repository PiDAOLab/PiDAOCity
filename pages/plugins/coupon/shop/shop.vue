<template>
    <view :class="theme_view">
        <view class="page-bottom-fixed">
            <!-- 优惠劵列表 -->
            <view v-if="data_list.length > 0" class="plugins-coupon-container padding-horizontal-main padding-top-main">
                <block v-for="(item, index) in data_list" :key="index">
                    <view :class="'item border-radius-main bg-white spacing-mb ' + (item.is_operable == 0 ? 'item-disabled' : '')">
                        <view class="v-left fl">
                            <view class="base single-text" :style="'color:' + item.bg_color_value + ';'">
                                <text v-if="item.type == 0" class="symbol">{{ currency_symbol }}</text>
                                <text class="price">{{ item.discount_value }}</text>
                                <text class="unit">{{ item.type_unit }}</text>
                                <text v-if="(item.desc || null) != null" class="desc cr-grey">{{ item.desc }}</text>
                            </view>
                            <view v-if="(item.use_limit_type_name || null) != null" class="base-tips cr-base single-text text-size-xs">{{ item.use_limit_type_name }}</view>
                        </view>
                        <view class="v-right fr cp" @tap="coupon_receive_event" :data-index="index" :data-value="item.id" :style="'background:' + item.bg_color_value + ';'">
                            <text class="circle"></text>
                            <text>{{ item.is_operable_name }}</text>
                        </view>
                    </view>
                </block>
            </view>
            <view v-else>
                <!-- 提示信息 -->
                <component-no-data :propStatus="data_list_loding_status" :propMsg="data_list_loding_msg"></component-no-data>
            </view>

            <!-- 结尾 -->
            <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>

            <!-- 回到店铺 -->
            <view v-if="(shop || null) != null" class="bottom-fixed">
                <view class="bottom-line-exclude">
                    <button class="bg-main br-main cr-white round dis-block" type="default" hover-class="none" size="mini" @tap="shop_event" :data-value="shop.url">
                        <view class="dis-inline-block va-m">
                            <uni-icons type="shop" size="16" color="#fff"></uni-icons>
                        </view>
                        <text class="va-m margin-left-sm">{{$t('index.index.i78v36')}}</text>
                    </button>
                </view>
            </view>
        </view>
    </view>
</template>
<script>
    const app = getApp();
    import componentNoData from '../../../../components/no-data/no-data';
    import componentBottomLine from '../../../../components/bottom-line/bottom-line';

    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                data_bottom_line_status: false,
                data_list_loding_status: 1,
                data_list_loding_msg: '',
                currency_symbol: app.globalData.currency_symbol(),
                params: null,
                data_list: [],
                data_base: null,
                shop: null,
                // 优惠劵领取
                temp_coupon_receive_index: null,
                temp_coupon_receive_value: null,
                // 自定义分享信息
                share_info: {},
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

            // 设置参数
            this.setData({
                params: params,
            });
        },

        onShow() {
            // 调用公共事件方法
            app.globalData.page_event_onshow_handle();

            // 数据加载
            this.init();

            // 初始化配置
            this.init_config();
        },

        // 下拉刷新
        onPullDownRefresh() {
            this.get_data_list();
        },

        methods: {
            // 初始化配置
            init_config(status) {
                if ((status || false) == true) {
                    this.setData({
                        currency_symbol: app.globalData.get_config('currency_symbol'),
                    });
                } else {
                    app.globalData.is_config(this, 'init_config');
                }
            },

            // 获取数据
            init() {
                this.get_data_list();
            },

            // 获取数据
            get_data_list() {
                var self = this;
                uni.showLoading({
                    title: this.$t('common.loading_in_text'),
                });
                if (self.data_list.length <= 0) {
                    self.setData({
                        data_list_loding_status: 1,
                    });
                }
                uni.request({
                    url: app.globalData.get_request_url('shop', 'index', 'coupon'),
                    method: 'POST',
                    data: this.params,
                    dataType: 'json',
                    success: (res) => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            var status = (data.data || []).length > 0;
                            this.setData({
                                data_base: data.base || null,
                                shop: data.shop || null,
                                data_list: data.data || [],
                                data_list_loding_msg: '',
                                data_list_loding_status: status ? 3 : 0,
                                data_bottom_line_status: status,
                            });

                            if ((this.data_base || null) != null) {
                                // 基础自定义分享
                                this.setData({
                                    share_info: {
                                        title: this.data_base.seo_title || this.data_base.application_name,
                                        desc: this.data_base.seo_desc,
                                        path: '/pages/plugins/coupon/shop/shop',
                                    },
                                });

                                // 导航名称
                                if ((this.data_base.shop_application_name || null) != null) {
                                    uni.setNavigationBarTitle({
                                        title: this.data_base.shop_application_name,
                                    });
                                }
                            }
                        } else {
                            self.setData({
                                data_bottom_line_status: false,
                                data_list_loding_status: 2,
                                data_list_loding_msg: res.data.msg,
                            });
                            app.globalData.showToast(res.data.msg);
                        }

                        // 分享菜单处理
                        app.globalData.page_share_handle(this.share_info);
                    },
                    fail: () => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        self.setData({
                            data_bottom_line_status: false,
                            data_list_loding_status: 2,
                            data_list_loding_msg: this.$t('common.internet_error_tips'),
                        });
                        app.globalData.showToast(this.$t('common.internet_error_tips'));
                    },
                });
            },

            // 优惠劵领取事件
            coupon_receive_event(e) {
                if (!app.globalData.is_single_page_check()) {
                    return false;
                }
                // 参数处理
                if ((e || null) == null) {
                    var index = this.temp_coupon_receive_index;
                    var value = this.temp_coupon_receive_value;
                } else {
                    var index = e.currentTarget.dataset.index;
                    var value = e.currentTarget.dataset.value;
                    this.setData({
                        temp_coupon_receive_index: index,
                        temp_coupon_receive_value: value,
                    });
                }

                // 登录校验
                var user = app.globalData.get_user_info(this, 'coupon_receive_event');
                if (user != false) {
                    // 用户未绑定手机则转到登录页面
                    if (app.globalData.user_is_need_login(user)) {
                        uni.navigateTo({
                            url: '/pages/login/login?event_callback=coupon_receive_event',
                        });
                        return false;
                    } else {
                        var temp_list = this.data_list;
                        if (temp_list[index]['is_operable'] != 0) {
                            uni.showLoading({
                                title: this.$t('common.processing_in_text'),
                            });
                            uni.request({
                                url: app.globalData.get_request_url('receive', 'coupon', 'coupon'),
                                method: 'POST',
                                data: {
                                    coupon_id: value,
                                },
                                dataType: 'json',
                                success: (res) => {
                                    uni.hideLoading();
                                    if (res.data.code == 0) {
                                        app.globalData.showToast(res.data.msg, 'success');
                                        if (this.data_base != null && this.data_base.is_repeat_receive != 1) {
                                            temp_list[index]['is_operable'] = 0;
                                            temp_list[index]['is_operable_name'] = this.$t('shop.shop.4q9oe2');
                                            this.setData({
                                                data_list: temp_list,
                                            });
                                        }
                                    } else {
                                        if (app.globalData.is_login_check(res.data, this, 'coupon_receive_event')) {
                                            app.globalData.showToast(res.data.msg);
                                        }
                                    }
                                },
                                fail: () => {
                                    uni.hideLoading();
                                    app.globalData.showToast(this.$t('common.internet_error_tips'));
                                },
                            });
                        }
                    }
                }
            },

            // 店铺事件
            shop_event(e) {
                var prev_url = app.globalData.prev_page();
                if (prev_url != null && prev_url.indexOf('pages/plugins/shop/detail/detail') != -1) {
                    uni.navigateBack();
                } else {
                    app.globalData.url_event(e);
                }
            },
        },
    };
</script>
<style></style>
