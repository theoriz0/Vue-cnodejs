<template>
    <div style="height: 100%">
        <nv-head page-type="消息" :fix-head="true" :show-menu="showMenu" :message-count="no_read_len" :need-add="false">
        </nv-head>
        <div class="page">
            <ul class="tabs">
                <li class="item br" :class='{"selected":selectItem === 1}' @click="changeItem(1)">
                    未读消息
                    <i class="iconfont read" v-show="no_read_len > 0" @click="markall">全标已读</i>
                </li>
                <li class="item" :class='{"selected":selectItem === 2}' @click="changeItem(2)">已读消息</li>
            </ul>
            <div class="message" v-for="(item, idx) in currentData">
                <router-link :to="{name:'topic',params:{id:item.topic_id}}">
                    <section class="user">
                        <div class="info">
                            <span class="cl">
                                {{item.content}}
                            </span>
                            <span class="cr">
                                <span class="name" v-text="getLastTimeStr(item.created_at, true)"></span>
                            </span>
                        </div>
                    </section>
                </router-link>
            </div>


            <div class="no-data" v-show="noData">
                <i class="iconfont icon-empty">&#xe60a;</i>
                暂无数据!
            </div>
        </div>
    </div>
</template>
<script>
    import $ from 'webpack-zepto';
    import utils from '../libs/utils.js';
    import nvHead from '../components/header.vue';
    import {
        mapGetters
    } from 'vuex';

    export default {
        data() {
            return {
                showMenu: false,
                selectItem: 2,
                message: {},
                noData: false,
                currentData: [],
                no_read_len: 0
            };
        },
        computed: {
            ...mapGetters({
                userInfo: 'getUserInfo'
            })
        },
        mounted() {
            $.get('https://cnodejs.org/api/v1/messages?accesstoken=' + this.userInfo.token, (d) => {
                if (d && d.data) {
                    this.message = d.data;
                    this.no_read_len = d.data.hasnot_read_messages.length;
                    if (d.data.hasnot_read_messages.length > 0) {
                        this.currentData = d.data.hasnot_read_messages;
                    } else {
                        this.currentData = d.data.has_read_messages;
                        this.selectItem = 2;
                    }
                    this.noData = this.currentData.length === 0;
                } else {
                    this.noData = true;
                }
            });
        },
        methods: {
            // 切换tab
            changeItem(idx) {
                this.selectItem = idx;
                this.currentData = idx === 1 ? this.message.hasnot_read_messages : this.message.has_read_messages;
                this.noData = this.currentData.length === 0;
            },
            // 标记所有为已读
            markall() {
                $.post('https://cnodejs.org/api/v1/message/mark_all', {
                    accesstoken: this.userInfo.token
                }, (d) => {
                    if (d && d.success) {
                        window.location.reload();
                    }
                });
            },
            getLastTimeStr(date, friendly) {
                return utils.getLastTimeStr(date, friendly);
            }
        },
        components: {
            nvHead
        }
    };
</script>
