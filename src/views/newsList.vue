<template>
    <div ref="wrap">
        <!-- 新闻列表 -->
        <div class="news-list">
            <div class="news-list-title flex">
                <text class="f28 fw5 c0">{{i18n.new}}</text>
            </div>
            <div v-if='isShow'>
                <div class="new-list-content" v-if='newsList.length != 0'>
                    <div v-for="(item,index) in newsList" :key='index' @click='newsListItemEvent(item.action)'>
                        <div class="flex-dr content-item" v-if='item.image !=""'>
                            <div class='content-item-left'>
                                <bui-image @click='newsListItemEvent(item.action)' placeholder='/image/ellipsis.png' :src="item.image" radius='10' width="100wx" height="73wx"></bui-image>
                            </div>
                            <div class='content-item-right flex-sb'>
                                <text class="item-right-text lines2 f28 c0 fw4">{{item.title}}</text>
                                <div class="item-right-date flex">
                                    <div class="date-origin flex">
                                        <text class="f24 c9">{{item.time}}</text>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div v-if='item.image ==""' class="flex-sb flex-dr flex-ac content-no-image">
                            <text class="f28 fw5 c0 lines1 flex10">{{item.title}}</text>
                            <text class="f24 c9 flex2 tr">{{item.time}}</text>
                        </div>
                    </div>
                </div>
                <div class="no-content flex-ac flex-jc" v-else>
                    <div class="flex-dr">
                        <bui-image src="/image/nodata.png" width="20wx" height="20wx"></bui-image>
                        <text class="f26 c51 fw4 pl15 center-height">{{isError?i18n.NoneData:i18n.ErrorLoadData}}</text>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
const link = weex.requireModule("LinkModule");
const linkapi = require("linkapi");
const dom = weex.requireModule("dom");
const storage = weex.requireModule('storage');
var uamFileIdUiDownload = '/ui/download?fileId=${id}&access=anonymous';
export default {
    data() {
        return {
            newsList: [],
            channel: new BroadcastChannel("WidgetsMessage"),
            i18n: "",
            isShow: false,
            isError: true,
        };
    },
    methods: {
        // 新闻列表
        newsListItemEvent(url) {
            if (url) {
                linkapi.openLinkBroswer("", url);
            } else {
                this.$toast('地址不存在');
            }
        },
        timestampToTime(timestamp) {
            var date = new Date(timestamp);
            var Y = date.getFullYear() + "-";
            var M =
                (date.getMonth() + 1 < 10
                    ? "0" + (date.getMonth() + 1)
                    : date.getMonth() + 1) + "-";
            var D = date.getDate() + " ";
            var h = date.getHours() + ":";
            var m = date.getMinutes() + ":";
            var s = date.getSeconds();
            return M + D;
        },
        UrlAddParam(url, name, value) {
            var r = url;
            if (r != null && r != 'undefined' && r != "") {
                value = encodeURIComponent(value);
                var reg = new RegExp("(^|)" + name + "=([^&]*)(|$)");
                var tmp = name + "=" + value;
                if (url.match(reg) != null) {
                    r = url.replace(eval(reg), tmp);
                } else {
                    if (url.match("[?]")) {
                        r = url + "&" + tmp;
                    } else {
                        r = url + "?" + tmp;
                    }
                }
            }
            return r;
        },
        getUrl(actionParams) {
            if (!actionParams) return {};
            var sPkey = /\s/;
            var us = actionParams.split(sPkey);
            var params = {};
            for (var i = 0; i < us.length; i++) {
                var u = us[i];
                var idx = u.indexOf('=');
                if (idx > -1) {
                    params[u.substring(0, idx).trim()] = u.substring(idx + 1, u.length).trim();
                }
            }
            var url = params.url;
            delete params.url;
            for (var key in params) {
                url = this.UrlAddParam(url, key, params[key]);
            }
            return url;
        },
        getAction(action) {
            if (typeof action === 'string') {
                action = action.replace(/[\r\n]/g, ' ')
                try {
                    action = JSON.parse(action)
                } catch (e) {
                    action = eval('(' + action + ')');
                }
            }
            var pcAction = action.pc || action.android;
            if (pcAction.indexOf('[OpenUrl]') === 0) {
                return this.getUrl(pcAction);
            } else {
                return ''
            }
        },
        getStorage(callback) {
            storage.getItem('newListJLocalData', res => {
                if (res.result == 'success') {
                    var data = JSON.parse(res.data)
                    this.isShow = true
                    this.isError = true
                    this.newsList = data;
                    this.broadcastWidgetHeight()
                } else {
                    callback()
                }
            })
        },
        getNewsData() {
            link.getServerConfigs([], params => {
                linkapi.get({
                    url: params.comwidgetsUri + "/news/list",
                    data: {
                        limit: 5
                    }
                }).then(res => {
                    this.isShow = true
                    this.isError = true
                    if (res.code == 200) {
                        try {
                            let newsArr = [];
                            for (let index = 0; index < res.data.length; index++) {
                                const element = res.data[index];
                                let newsItemObj = {};
                                let action = this.getAction(element.action)
                                newsItemObj["action"] = action
                                newsItemObj["title"] = element.title;
                                newsItemObj["time"] = this.timestampToTime(element.publishTime);
                                newsItemObj["image"] = this.parseImgUrl(element.image, params.uamUri || params.uamUrl);
                                newsArr.push(newsItemObj);
                            }
                            this.newsList = newsArr;
                            this.broadcastWidgetHeight();
                            storage.setItem('newListJLocalData', JSON.stringify(newsArr))
                        } catch (error) {
                            this.error();
                        }
                    }
                }, err => {
                    this.error();
                }
                );
            }, err => {
                this.error();
            });
        },
        parseImgUrl(url, uam) {
            if (!url) return url;
            if (url.match(/^https?:\/\//g)) {
                return url
            }
            if (url.startsWith("store://")) {
                url = url.replace("store://", "");
            }
            url = (uam + uamFileIdUiDownload).replace('${id}', url);
            return url;
        },
        error() {
            this.isError = false
            this.isShow = true
            this.broadcastWidgetHeight();
        },
        getComponentRect(_params) {
            dom.getComponentRect(this.$refs.wrap, (ret) => {
                this.channel.postMessage({
                    widgetHeight: ret.size.height,
                    id: _params.id
                });
            });
        },
        broadcastWidgetHeight() {
            let _params = this.$getPageParams();
            // 防止高度通知失败
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 200)
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 1200)
        }
    },
    created() {
        this.$fixViewport();
        linkapi.getLanguage(res => {
            this.i18n = this.$window[res];
        });
    },
    mounted() {
        var that = this
        this.channel.onmessage = event => {
            if (event.data.action === "RefreshData") {
                this.getNewsData();
            }
        };
        this.getStorage(function () {
            that.getNewsData()
        })
    }
};
</script>

<style lang="css" src="../css/common.css"></style>
<style>
.news-list {
    background-color: #fff;
}

.news-list-title {
    height: 44wx;
    padding: 0 12wx;
    border-bottom: 1px solid #f2f2f2;
}

.content-item {
    margin: 13wx 11wx 13wx 12wx;
}

.content-item-left {
    width: 200px;
}

.content-item-right {
    margin-left: 10wx;
    flex: 1;
    flex-direction: column;
}

.item-right-text {
    flex: 1;
    color: rgb(165, 164, 164);
}

.item-right-date {
    padding-top: 10wx;
}

.date-origin {
    width: 200px;
}

.content-no-image {
    height: 40wx;
    margin: 9wx 11wx 5wx 12wx;
}

.no-content {
    height: 200wx;
    flex: 1;
}
</style>