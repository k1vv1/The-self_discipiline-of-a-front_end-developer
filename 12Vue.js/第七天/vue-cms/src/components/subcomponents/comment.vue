<template>
    <div class="cmt-container">
        <h3>发表评论</h3>
        <hr>
        <textarea placeholder="请输入要BB的内容(最多BB120个字)" maxlength="120" v-model="msg"></textarea>

        <mt-button type="primary" size="large" @click="postComment">发表评论</mt-button>

        <div class="cmt-list">
            <div class="cmt-item" v-for="(item, i) in comments" :key="item.add_time">
                <div class="cmt-title">
                    第{{i+1}}楼&nbsp;&nbsp;用户：{{item.user_name}}&nbsp;&nbsp;发表时间：{{item.add_time | dateFormat}}
                </div>
                <div class="cmt-body">
                    {{item.content === 'undefined' ? '此用户很懒，嘛都没说' : item.content}}
                </div>
            </div>
        </div>

        <mt-button type="danger" size="large" plain @clcik="getMore">加载更多</mt-button>
    </div>
</template>


<script>
import { Toast } from 'mint-ui'

export default {
    data(){
        return {
            pageindex: 1,  // 默认展示第一页
            comments: [],  // 所有的评论数据
            msg: '' //评论输入的内容
        }
    },
    created(){
        this.getComments();
    },
    methods: {
        getComments(){
            this.$http.get('api/getcomments/'+this.id+'?pageindex='+this.pageindex).then(result => {
                if(result.body.status === 0){
                    // this.comments = result.body.message;
                    // 每当获取新评论数据的时候，不要把老数据覆盖，而是拼接上新数据
                    this.comments = this.comments.concat(result.body.message);
                }else{
                    Toast('获取评论失败！')
                }
            })
        },
        getMore(){
            this.pageindex++;
            this.getComments()
        },
        postComment(){
            // 校验是否为空内容
            if(this.msg.trim().length === 0){
                return Toast('评论内容不能为空！');
            }


            // 发表评论
            // 参数1：请求的URL地址
            // 参数2：提交给服务器的数据对象{content: 'this.msg'}
            // 参数3：定义提交的时候，表单中数据的格式 {emulateJSON: true}
            this.$http.post('api/postcomment/' + this.$route.params.id, {
                content: this.msg.trim()
            })
            .then(function(result){
                if(result.body.status === 0){
                    var cmt = {
                        user_name: '匿名用户',
                        add_time: Date.now(),
                        content: this.msg.trim()
                    };
                    this.comments.unshift(cmt);
                    this.msg = '';
                }
            });
        }
    },
    props: ["id"]
}
</script>

<style lang="scss" scoped>
    .cmt-container{
        h3{
            font-size: 18px;
        }
        textarea{
            font-size: 16px;
            height: 85px;
            margin: 0;
        }
        cmt-list{
            margin: 5px 0;
            cmt-item{
                font-size: 13px;
               cmt-title{
                   background-color: #ccc;
                   line-height: 30px;
               } 
               cmt-body{
                   line-height: 35px;
                   text-indent: 2em;
               }
            }
        }
    }
</style>