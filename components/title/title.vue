<template>
    <view :class="theme_view">
        <view class="spacing-nav-title flex-row align-c jc-sb text-size-xs">
            <view class="title-left">
                <text class="text-wrapper" :class="propTitleLeftBorder ? 'title-left-border' : ''" :style="'color:' + propTitleColor + ';'">{{propTitle}}</text>
                <text class="vice-name margin-left-sm" :style="'color:' + propViceTitleColor + ';'">{{propViceTitle}}</text>
            </view>
            <navigator :url="propMoreUrl" hover-class="none" class="arrow-right padding-right cr-grey">{{propMore || this.$t('common.more')}}</navigator>
        </view>
    </view>
</template>

<script>
    const app = getApp();
    export default {
        props: {
            propTitle: {
                type: String,
                default: ''
            },
            propTitleColor: {
                type: String,
                default: '#333'
            },
            propViceTitle: {
                type: String,
                default: ''
            },
            propViceTitleColor: {
                type: String,
                default: '#999'
            },
            propMore: {
                type: String,
                default: '',
            },
            propMoreUrl: {
                type: String,
                default: ''
            },
            propTitleLeftBorder: {
                type: Boolean,
                default: true
            },
        },
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
            };
        }
    }
</script>

<style>

</style>