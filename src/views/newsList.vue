<template>
    <div ref="wrap" class="main">
        <!-- 新闻列表 -->
        <div class="news-list">
            <div class="news-list-title flex">
                <text class="f28 fw5 c0">{{i18n.new}}</text>
            </div>
            <div class="new-list-content">
                <div v-for="(item,index) in newsList" :key='index' @click='newsListItemEvent(item.action)'>
                    <div class="flex-dr content-item" v-if='item.image !=""'>
                        <div  class='content-item-left'>
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
        </div>
    </div>
</template>

<script>
    const link = weex.requireModule("LinkModule");
    const linkapi = require("linkapi");
    const dom = weex.requireModule("dom");
    export default {
        data() {
            return {
                newsList: [],
                channel: new BroadcastChannel("WidgetsMessage"),
                i18n: "",
                URL_PATTERN: /((http(s)?:\/\/|www\.|WWW\.)([/\w-./@?_!~$%&=:#;+\-()]*)?)/g,
                STORE_PATTERN: /(store:\/\/)/g
            };
        },
        methods: {
            // 新闻列表
            newsListItemEvent(url) {
                linkapi.openLinkBroswer("", url);
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
            getNewsData() {
                this.getToken(
                    token => {
                        link.getServerConfigs([], params => {
                            linkapi.get({
                                url: params.comwidgetsUri + "/news/list"
                            }).then(res => {
                                if (res.code == 200) {
                                    let newsArr = [];
                                    for (let index = 0; index < res.data.length; index++) {
                                        const element = res.data[index];
                                        let newsItemObj = {};
                                        let action = JSON.parse(element.action);
                                        newsItemObj["action"] = action.mobile_web ? action.mobile_web : action.web;
                                        newsItemObj["title"] = element.title;
                                        newsItemObj["time"] = this.timestampToTime(element.publishTime);
                                        newsItemObj["image"] = "";
                                        if (element.image) {
                                            newsItemObj["image"] = this.getImageUrl(element.image, token.accessToken, params.storeUri);
                                            // newsItemObj["image"] = newsItemObj["image"] + "&size=0x71&access_token=" + token.accessToken;
                                        }
                                        newsArr.push(newsItemObj);
                                    }
                                    this.newsList = newsArr;
                                    this.broadcastWidgetHeight();
                                }
                            }, err => {
                                this.error();
                            }
                            );
                        }, err => {
                            this.error();
                        }
                        );
                    }, err => {
                        this.error();
                    }
                );
            },
            getToken(success, error) {
                return new Promise((resolve, reject) => {
                    link.getToken([], res => {
                        resolve(res);
                        success && success(res);
                    }, err => {
                        reject(err);
                        error && error(err);
                    }
                    );
                });
            },
            getLastKeyStr(str, key) {
                if (!str) return "";
                return str.substring(str.lastIndexOf(key) + 1, str.length);
            },
            error() {
                this.broadcastWidgetHeight();
            },
            getImageUrl(resourceUrl, accessToken, storeUri) {
                var resourceAtUrl = '';
                var storeFileId;
                var storeGetFile = storeUri + '/store/getFile?fileId=';
                if (resourceUrl.match(this.STORE_PATTERN)) {
                    if (resourceUrl.length > 20) {
                        storeFileId = resourceUrl.replace(this.STORE_PATTERN, "");
                        resourceAtUrl = storeGetFile + storeFileId + '&access_token=' + accessToken;
                    }
                } else if (resourceUrl.match(this.URL_PATTERN)) {
                    resourceAtUrl = resourceUrl;
                } else {
                    storeFileId = resourceUrl.replace(/(^.+fileId=)/g, "");
                    resourceAtUrl = storeGetFile + storeFileId + '&access_token=' + accessToken;
                }
                return resourceAtUrl;
            },
            broadcastWidgetHeight() {
                let _params = this.$getPageParams();
                setTimeout(() => {
					dom.getComponentRect(this.$refs.wrap, (ret) => {
						this.channel.postMessage({
							widgetHeight: ret.size.height,
							id: _params.id
						});
					});
				}, 200)
            }
        },
        created() {
            this.$fixViewport();
            linkapi.getLanguage(res => {
                this.i18n = this.$window[res];
            });
        },
        mounted() {
            this.channel.onmessage = event => {
                if (event.data.action === "RefreshData") {
                    this.getNewsData();
                }
            };
            this.getNewsData();
        }
    };
</script>

<style lang="css" src="../css/common.css"></style>
<style>
    .main {
        flex: 1;
    }
    .news-list {
        background-color: #fff;
    }

    .news-list-title {
        height: 20wx;
        margin: 9wx 11wx 5wx 12wx;
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
</style>