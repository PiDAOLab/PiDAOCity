<template>
    <view :class="theme_view">
        <view v-if="(data || null) != null" class="pr">
            <!-- 头部 -->
            <block v-if="(data.is_header || 0) == 1">
                <!-- 搜索 -->
                <view :class="'padding-main bg-white pr oh br-b search '+(is_shop_search_all_search_button == 1 ? '' : 'header-shop-whole-search')">
                    <input class="bg-white fl padding-left-xxl text-size-xs round border-color-main" type="done" :placeholder="$t('design.design.ses9m2')" :value="search_keywords_value || ''" placeholder-class="cr-grey" @input="search_keywords_event">
                    <view class="search-btn pa">
                        <button class="bg-main br-main cr-white round text-size-xs" type="default" size="mini" hover-class="none" @tap="search_button_event" :data-value="'/pages/plugins/shop/search/search?shop_id='+shop.id+'&'">{{is_shop_search_all_search_button == 1 ? $t('design.design.i7725u') : $t('common.search')}}</button>
                        <button v-if="is_shop_search_all_search_button == 1" class="bg-main-pair br-main-pair cr-white round text-size-xs" type="default" size="mini" hover-class="none" @tap="search_button_event" data-value="/pages/goods-search/goods-search?">{{$t('design.design.ay7m42')}}</button>
                    </view>
                </view>
                <!-- 顶部 -->
                <view class="header plugins-shop-data-list bg-white oh">
                    <image :src="shop.logo" mode="widthFix" class="shop-logo fl border-radius-main cp"></image>
                    <view class="base fr item">
                        <view class="shop-title single-text">
                            <!-- 认证信息 -->
                            <view v-if="(data_base.is_enable_auth || 0) == 1 && ((shop.auth_type != -1 && (shop.auth_type_msg || null) != null) || ((shop.bond_status || 0) == 1 && (shop.bond_status_msg || null) != null))" class="auth-icon dis-inline-block">
                                <!-- 实名认证 -->
                                <block v-if="shop.auth_type != -1 && (shop.auth_type_msg || null) != null">
                                    <image :src="shop.auth_type == 0 ? data_base.shop_auth_personal_icon : data_base.shop_auth_company_icon" class="icon va-m" mode="aspectFill" :data-value="'/pages/plugins/shop/license/license?id='+shop.id" @tap="url_event"></image>
                                </block>
                                <!-- 保证金认证 -->
                                <block v-if="(shop.bond_status || 0) == 1 && (shop.bond_status_msg || null) != null">
                                    <image :src="data_base.shop_auth_bond_icon" class="icon va-m" mode="aspectFill"></image>
                                </block>
                            </view>
                            <!-- 标题 -->
                            <text class="fw-b text-size va-m">{{shop.name}}</text>
                        </view>
                        <view class="base-bottom oh margin-top-sm text-size-xs">
                            <!-- 在线客服 -->
                            <view v-if="(data_base.is_service_info || 0) == 1" class="fl margin-right-xxl cp" @tap="header_service_event">
                                <image class="va-m margin-right-sm" :src="common_static_url+'customer-service-icon.png'" mode="scaleToFill"></image>
                                <text class="va-m cr-base">{{$t('design.design.21kak7')}}</text>
                            </view>
                            <!-- 收藏 -->
                            <view class="fl single-text cp" @tap="shop_favor_event">
                                <image class="va-m margin-right-sm" :src="common_static_url+'favor'+(shop_favor_info.status == 1 ? '-active' : '')+'-icon.png'" mode="scaleToFill"></image>
                                <text :class="'va-m ' + (shop_favor_info.status == 1 ? 'cr-main' : '')">{{shop_favor_info.text}}({{shop_favor_info.count}})</text>
                            </view>
                            <!-- 评分 -->
                            <view v-if="(shop.score_data || null) != null" class="fl margin-left-xxl">
                                <view class="dis-inline-block va-m">
                                    <uni-rate :value="shop.score_data.value" :readonly="true" :is-fill="false" :size="14" />
                                </view>
                                <text class="va-m cr-red">{{shop.score_data.value}}{{$t('design.design.745kx2')}}</text>
                            </view>
                        </view>
                    </view>
                </view>
                <!-- 在线客服 -->
                <view v-if="header_service_status && ((data_base.is_service_info || 0) == 1 || (shop.chat_info || null) != null)" class="header-service pa border-radius-main oh bg-white br">
                    <view v-if="(shop.chat_info || null) != null" class="item padding-main br-t single-text">
                        <text class="va-m">{{$t('detail.detail.r4124d')}}</text>
                        <view class="dis-inline-block chat-info cp" @tap="chat_event">
                            <image class="dis-inline-block va-m" :src="shop.chat_info.icon" mode="scaleToFill"></image>
                            <text class="margin-left-sm va-m cr-blue" :data-value="shop.chat_info.chat_url">{{shop.chat_info.name}}</text>
                        </view>
                    </view>
                    <view v-if="(shop.service_qq || null) != null" class="item padding-main br-t single-text">
                        <text>Q Q：</text>
                        <text class="cp" @tap="text_copy_event" :data-value="shop.service_qq">{{shop.service_qq}}</text>
                    </view>
                    <view v-if="(shop.service_tel || null) != null" class="item padding-main br-t single-text">
                        <text>{{$t('order.order.7dxbm5')}}</text>
                        <text class="cp" @tap="tel_event" :data-value="shop.service_tel">{{shop.service_tel}}</text>
                    </view>
                    <view v-if="(shop.open_week_name || null) != null && (shop.close_week_name || null) != null" class="item padding-main br-t single-text">
                        <text>{{$t('article-detail.article-detail.728374')}}</text>
                        <text class="cp" @tap="text_copy_event" :data-value="shop.open_week_name + $t('design.design.gv16tj') + shop.close_week_name + '，' + shop.open_time + '-' + shop.close_time">{{shop.open_week_name}}{{$t('design.design.882220')}}{{shop.close_time}}</text>
                    </view>
                    <view v-if="(shop.service_weixin_qrcode || null) != null || (shop.service_line_qrcode || null) != null" class="oh qrcode tc br-t padding-top-main">
                        <view v-if="(shop.service_weixin_qrcode || null) != null" class="item padding-bottom-lg dis-inline-block">
                            <image class="radius cp" :src="shop.service_weixin_qrcode" mode="scaleToFill" @tap="image_show_event" :data-value="shop.service_weixin_qrcode"></image>
                            <view>{{$t('detail.detail.54k10s')}}</view>
                        </view>
                        <view v-if="(shop.service_line_qrcode || null) != null" class="item padding-bottom-lg dis-inline-block">
                            <image class="radius cp" :src="shop.service_line_qrcode" mode="scaleToFill" @tap="image_show_event" :data-value="shop.service_line_qrcode"></image>
                            <view>{{$t('detail.detail.vj4nom')}}</view>
                        </view>
                    </view>
                </view>
                <!-- 导航 -->
                <view v-if="shop_goods_category.length > 0 || shop_navigation.length > 0" class="nav scroll-view-horizontal bg-white padding-top-lg border-color-main">
                    <view v-if="shop_goods_category.length > 0" class="item padding-main arrow-bottom nav-shop-category dis-inline-block fw-b cp" @tap="nav_shop_category_event">{{$t('design.design.wtx1l8')}}</view>
                    <scroll-view scroll-x class="nav-scroll">
                        <block v-for="(item, index) in shop_navigation" :key="index">
                            <block v-if="(item.items || null) == null || item.items.length == 0">
                                <view class="item dis-inline-block fw-b cp" @tap="nav_event" :data-value="item.url" :data-index="index">{{item.name}}</view>
                            </block>
                            <block v-else>
                                <view class="item dis-inline-block fw-b cp" @tap="nav_event" :data-index="index">{{item.name}}</view>
                                <view v-if="(item.items_status || 0) == 1" class="nav-items pf border-radius-main oh bg-white br">
                                    <block v-for="(items, index2) in item.items" :key="index2">
                                        <view class="item fw-b cp margin-vertical-main" @tap="nav_event" :data-value="items.url" :data-index="index" :data-indexs="index2">{{items.name}}</view>
                                    </block>
                                </view>
                            </block>
                        </block>
                    </scroll-view>
                    <view v-if="nav_category_status" class="nav-category bg-white pa tc">
                        <scroll-view scroll-y class="category-scroll">
                            <block v-if="shop_goods_category.length > 0">
                                <block v-for="(item, index) in shop_goods_category" :key="index">
                                    <view class="item dis-block cr-base single-text cp" @tap="shop_category_event" :data-value="item.id">{{item.name}}</view>
                                </block>
                            </block>
                            <block v-else>
                                <view class="padding-top-xxl padding-bottom-xxl cr-grey">{{$t('design.design.83occ4')}}</view>
                            </block>
                        </scroll-view>
                    </view>
                </view>
            </block>

            <!-- 拖拽模式、引入拖拽数据模块 -->
            <component-layout :propData="layout_data"></component-layout>

            <!-- 结尾 -->
            <block v-if="(data.is_footer || 0) == 1">
                <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>
            </block>
        </view>
        <view v-else>
            <!-- 提示信息 -->
            <component-no-data :propStatus="data_list_loding_status" :propMsg="data_list_loding_msg"></component-no-data>
        </view>
    </view>
</template>
<script>
    const app = getApp();
    import componentLayout from "../../../../components/layout/layout";
    import componentNoData from "../../../../components/no-data/no-data";
    import componentBottomLine from "../../../../components/bottom-line/bottom-line";

    var common_static_url = app.globalData.get_static_url('common');
    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                common_static_url: common_static_url,
                is_shop_search_all_search_button: 0,
                data_bottom_line_status: false,
                data_list_loding_status: 1,
                data_list_loding_msg: '',
                params: null,
                user: null,
                data_base: null,
                shop: null,
                shop_favor_user: [],
                shop_navigation: [],
                shop_goods_category: [],
                data: null,
                layout_data: [],
                // 基础配置
                search_keywords_value: '',
                header_service_status: false,
                nav_category_status: false,
                shop_category_tab_value: 0,
                shop_favor_info: {
                    "text": this.$t('goods-detail.goods-detail.dco1sc'),
                    "status": 0,
                    "count": 0
                },
                // 自定义分享信息
                share_info: {}
            };
        },

        components: {
            componentLayout,
            componentNoData,
            componentBottomLine
        },
        props: {},

        onLoad(params) {
            // 调用公共事件方法
            app.globalData.page_event_onload_handle(params);

            // 设置参数
            this.setData({
                params: params,
                user: app.globalData.get_user_cache_info()
            });
        },

        onShow() {
            // 调用公共事件方法
            app.globalData.page_event_onshow_handle();

            // 加载数据
            this.get_data();
        },

        // 下拉刷新
        onPullDownRefresh() {
            this.get_data();
        },

        methods: {
            // 获取数据
            get_data() {
                uni.request({
                    url: app.globalData.get_request_url("index", "design", "shop"),
                    method: 'POST',
                    data: {
                        "id": this.params.id || 0
                    },
                    dataType: 'json',
                    success: res => {
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            var base = data.base || null;
                            this.setData({
                                data_base: base,
                                shop: (data.shop || null) == null || data.shop.length <= 0 ? null : data.shop,
                                shop_favor_user: data.shop_favor_user || [],
                                shop_navigation: data.shop_navigation || [],
                                shop_goods_category: data.shop_goods_category || [],
                                is_shop_search_all_search_button: (base == null || parseInt(base.is_shop_search_all_search_button || 0) != 1) ? 0 : 1,
                                data: (data.data || null) != null && data.data.length != 0 ? data.data : null,
                                layout_data: data.layout_data || [],
                                data_list_loding_msg: '',
                                data_list_loding_status: 0,
                                data_bottom_line_status: true
                            });

                            // 收藏信息
                            if ((this.shop || null) != null) {
                                var status = this.shop_favor_user.indexOf(this.shop.id) != -1 ? 1 : 0;
                                this.setData({
                                    shop_favor_info: {
                                        "count": this.shop.shop_favor_count || 0,
                                        "status": status,
                                        "text": (status == 1 ? this.$t('goods-detail.goods-detail.by7052') : '') + this.$t('goods-detail.goods-detail.dco1sc')
                                    }
                                });
                            }

                            if ((this.data || null) != null) {
                                // 基础自定义分享
                                this.setData({
                                    share_info: {
                                        title: this.data.seo_title || this.data.name,
                                        desc: this.data.seo_desc,
                                        path: '/pages/plugins/shop/design/design',
                                        query: 'id='+this.data.id,
                                        img: this.data.logo
                                    }
                                });

                                // 标题名称
                                uni.setNavigationBarTitle({
                                    title: this.data.name
                                });
                            }
                        } else {
                            this.setData({
                                data_bottom_line_status: false,
                                data_list_loding_status: 2,
                                data_list_loding_msg: res.data.msg
                            });
                        }

                        // 分享菜单处理
                        app.globalData.page_share_handle(this.share_info);
                    },
                    fail: () => {
                        uni.stopPullDownRefresh();
                        this.setData({
                            data_bottom_line_status: false,
                            data_list_loding_status: 2,
                            data_list_loding_msg: this.$t('common.internet_error_tips')
                        });
                        app.globalData.showToast(this.$t('common.internet_error_tips'));
                    }
                });
            },

            // 店铺收藏事件
            shop_favor_event(e) {
                var user = app.globalData.get_user_info(this, 'shop_favor_event');
                if (user != false) {
                    // 用户未绑定手机则转到登录页面
                    if (app.globalData.user_is_need_login(user)) {
                        uni.navigateTo({
                            url: "/pages/login/login?event_callback=shop_favor_event"
                        });
                        return false;
                    } else {
                        uni.showLoading({
                            title: this.$t('common.processing_in_text')
                        });
                        uni.request({
                            url: app.globalData.get_request_url("favor", "shopfavor", "shop"),
                            method: 'POST',
                            data: {
                                "id": this.shop.id
                            },
                            dataType: 'json',
                            success: res => {
                                uni.hideLoading();
                                if (res.data.code == 0) {
                                    this.setData({
                                        shop_favor_info: res.data.data
                                    });
                                    app.globalData.showToast(res.data.msg, 'success');
                                } else {
                                    if (app.globalData.is_login_check(res.data, this, 'shop_favor_event')) {
                                        app.globalData.showToast(res.data.msg);
                                    }
                                }
                            },
                            fail: () => {
                                uni.hideLoading();
                                app.globalData.showToast(this.$t('common.internet_error_tips'));
                            }
                        });
                    }
                }
            },

            // 搜索输入事件
            search_keywords_event(e) {
                this.setData({
                    search_keywords_value: e.detail.value || ''
                });
            },

            // 搜索事件
            search_button_event(e) {
                var value = e.currentTarget.dataset.value || null;
                uni.navigateTo({
                    url: value + 'keywords=' + this.search_keywords_value || ''
                });
            },

            // 导航分类事件
            header_service_event(e) {
                this.setData({
                    header_service_status: !this.header_service_status
                });
            },

            // 导航分类事件
            nav_shop_category_event(e) {
                this.setData({
                    nav_category_status: !this.nav_category_status
                });
            },

            // 导航分类事件
            shop_category_event(e) {
                var value = e.currentTarget.dataset.value || null;
                uni.navigateTo({
                    url: '/pages/plugins/shop/search/search?shop_id=' + this.shop.id + '&category_id=' + value
                });
            },

            // 导航事件
            nav_event(e) {
                // 存在子级则做子级显示隐藏处理
                var value = e.currentTarget.dataset.value || null;
                if(value == null) {
                    var index = e.currentTarget.dataset.index;
                    var temp_nav = this.shop_navigation;
                    for(var i in temp_nav) {
                        if(i == index) {
                            temp_nav[i]['items_status'] = ((temp_nav[i]['items_status'] || 0) == 0) ? 1 : 0;
                        } else {
                            temp_nav[i]['items_status'] = 0;
                        }
                    }
                    this.setData({shop_navigation: temp_nav});
                } else {
                    app.globalData.url_event(e);
                }
            },

            // url事件
            url_event(e) {
                app.globalData.url_event(e);
            },

            // 剪切板
            text_copy_event(e) {
                app.globalData.text_copy_event(e);
            },

            // 电话
            tel_event(e) {
                app.globalData.call_tel(e.currentTarget.dataset.value || null);
            },

            // 图片预览
            image_show_event(e) {
                app.globalData.image_show_event(e);
            },
            
            // 进入客服系统
            chat_event() {
                app.globalData.chat_entry_handle(this.shop.chat_info.chat_url);
            }
        }
    };
</script>
<style scoped>
    @import './design.css';
</style>