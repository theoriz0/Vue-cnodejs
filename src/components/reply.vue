<template>

    <section class="reply">
        <textarea id="content" rows="8" class="text"
            :class="{'err':hasErr}"
            v-model="content"
            placeholder='请输入回复'>
        </textarea>
        <a class="button" @click="addReply">确定</a>
    </section>

</template>
<script>
    import $ from 'webpack-zepto';
    // const utils = require('../libs/utils');
    // const markdown = require('markdown').markdown;
    import {
        mapGetters
    } from 'vuex';

    export default {
        replace: true,
        props: ['topic', 'replyId', 'topicId', 'replyTo', 'show'],
        data() {
            return {
                hasErr: false,
                content: ''
            };
        },
        computed: {
            ...mapGetters({
                userInfo: 'getUserInfo'
            })
        },
        mounted() {
            if (this.replyTo) {
                this.content = `@${this.replyTo} `;
            }
        },
        methods: {
            addReply() {
                if (!this.content) {
                    this.hasErr = true;
                } else {
                    let time = new Date();
                    let postData = {
                        content: this.content
                    };
                    if (this.replyId) {
                        postData.reply_id = this.replyId;
                    }
                    $.ajax({
                        type: 'POST',
                        url: `http://localhost:8088/topic/${this.topicId}/replies`,
                        data: postData,
                        dataType: 'json',
                        headers: {
                            'Authorization': this.userInfo.token
                        },
                        success: (res) => {
                            if (res.success) {
                                this.topic.replies.push({
                                    id: res.reply_id,
                                    author: {
                                        loginname: this.userInfo.loginname,
                                        avatar_url: this.userInfo.avatarUrl
                                    },
                                    content: this.content,
                                    ups: [],
                                    created_at: time
                                });
                            }
                            this.content = '';
                            if (this.show) {
                                this.$emit('close');
                            }
                        },
                        error: (res) => {
                            var error = JSON.parse(res.responseText);
                            this.$alert(error.error_msg);
                            return false;
                        }
                    });
                }
            }
        }
    };
</script>
