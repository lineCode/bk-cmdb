/*
 * Tencent is pleased to support the open source community by making 蓝鲸 available.
 * Copyright (C) 2017-2018 THL A29 Limited, a Tencent company. All rights reserved.
 * Licensed under the MIT License (the "License"); you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at http://opensource.org/licenses/MIT
 * Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and limitations under the License.
 */

<template>
    <div class="detail-wrapper" v-if="isShow">
        <div class="detail-box">
            <form id="validate-event">
                <div class="form-item">
                    <label for="" class="label-name">
                        推送名称<span class="color-danger">*</span>
                    </label>
                    <div class="item-content">
                        <input type="text" class="bk-form-input" placeholder="请输入推送名称"
                            maxlength="20"
                            v-model.trim="tempEventData['subscription_name']"
                            data-vv-name="推送名称"
                            v-validate="'required'"
                        >
                        <span v-show="errors.has('推送名称')" class="color-danger">{{ errors.first('推送名称') }}</span>
                    </div>
                </div>
                <div class="form-item">
                    <label for="" class="label-name">
                        系统名称
                    </label>
                    <div class="item-content">
                        <input type="text" class="bk-form-input" placeholder="请输入系统名称"
                            v-model.trim="tempEventData['system_name']"
                        >
                    </div>
                </div>
                <div class="form-item">
                    <label for="" class="label-name">
                        URL<span class="color-danger">*</span>
                    </label>
                    <div class="item-content">
                        <input type="text" class="bk-form-input" placeholder="请输入URL"
                            v-model.trim="tempEventData['callback_url']"
                            v-validate="'required|http'"
                            data-vv-name="http"
                        >
                        <span v-show="errors.has('http')" class="color-danger">{{ errors.first('http') }}</span>
                    </div>
                    <bk-button class="fl" type="default" style="margin-left:10px" @click.prevent="testPush">测试推送</bk-button>
                </div>
                <div class="form-item">
                    <label for="" class="label-name">
                        成功确认方式<span class="color-danger">*</span>
                    </label>
                    <div class="item-content">
                        <label for="http" class="bk-form-radio bk-radio-small">
                            <input type="radio" name="confimType" id="http" value="httpstatus"
                                v-model="tempEventData['confirm_mode']"
                            >HTTP状态
                        </label>
                        <label for="reg" class="bk-form-radio bk-radio-small">
                            <input type="radio" name="confimType" id="reg" value="regular"
                                v-model="tempEventData['confirm_mode']"
                            >正则验证
                        </label>
                        <input type="text" class="bk-form-input" placeholder="请输入正则验证"
                            v-if="tempEventData['confirm_mode'] === 'regular'"
                            v-model.trim="tempEventData['confirm_pattern']['regular']"
                            data-vv-name="该字段"
                            v-validate="'required'"
                        >
                        <input type="text" class="bk-form-input number" placeholder="成功标志"
                            v-else
                            v-model.trim="tempEventData['confirm_pattern']['httpstatus']"
                            v-validate="{required: true, regex: /^[0-9]+$/}"
                            data-vv-name="该字段"
                        >
                        <span v-show="errors.has('该字段')" class="color-danger">{{ errors.first('该字段') }}</span>
                    </div>
                </div>
                <div class="form-item">
                    <label for="" class="label-name">
                        超时时间<span class="color-danger">*</span>
                    </label>
                    <div class="item-content length-short">
                        <input type="text" class="bk-form-input" placeholder="单位：秒"
                            v-model.trim="tempEventData['time_out']"
                            v-validate="{required: true, regex: /^[0-9]+$/}"
                            data-vv-name="超时时间"
                            maxlength="10"
                        ><span class="unit">S</span>
                        <div v-show="errors.has('超时时间')" class="color-danger">{{ errors.first('超时时间') }}</div>
                    </div>
                </div>
            </form>
            <div class="info"><span :class="{'text-danger': subscriptionFormError}">至少选择1个事件</span>，已选择 <span class="num">{{selectNum}}</span> 个</div>
            <ul class="event-wrapper">
                <li class="event-box clearfix"
                    :key="index"
                    v-for="(classify, index) in eventPushList">
                    <div class="event-title" :title="classify.name">
                        {{classify.name}}
                        <i class="bk-icon icon-angle-down fr" :class="{'up': classify.isHidden}" @click="toggleEventList(classify)"></i>
                    </div>
                    <transition name="slide">
                        <ul v-if="!classify.isHidden" :height="classify.children.length*32" :style="eventHeight(classify.children.length)">
                            <li v-for="(item, idx) in classify.children" :key="idx" class="event-item" >
                                <template v-if="classify.isDefault">
                                    <template v-if="item.id==='host'">
                                        <label for="" class="label-name" :title="item.name">{{item.name}}</label>
                                        <div class="options">
                                            <label for="hostall" class="bk-form-checkbox bk-checkbox-small">
                                                <input type="checkbox"
                                                value="hostall"
                                                :checked="tempEventData['subscription_form'][item.id].length == 2"
                                                id="hostall" @change="checkAll('host')"><i class="bk-checkbox-text" title="全选">全选</i>
                                            </label>
                                            <label for="hostcreate" class="bk-form-checkbox bk-checkbox-small">
                                                <input type="checkbox"
                                                v-model="tempEventData['subscription_form'][item.id]"
                                                value="hostcreate"
                                                id="hostcreate"><i class="bk-checkbox-text" title="新增主机">新增主机</i>
                                            </label>
                                            <label for="hostdelete" class="bk-form-checkbox bk-checkbox-small">
                                                <input type="checkbox"
                                                value="hostdelete"
                                                v-model="tempEventData['subscription_form'][item.id]"
                                                id="hostdelete"><i class="bk-checkbox-text" title="删除主机">删除主机</i>
                                            </label>
                                        </div>
                                    </template>
                                    <template v-if="item.id==='module'">
                                        <label for="" class="label-name" :title="item.name">{{item.name}}</label>
                                        <div class="options">
                                            <label :for="'moduleall'" class="bk-form-checkbox bk-checkbox-small">
                                                <input type="checkbox"
                                                :value="'moduleall'"
                                                :checked="tempEventData['subscription_form'][item.id].length == 2"
                                                :id="'moduleall'" @change="checkAll('module')"><i class="bk-checkbox-text" title="全选">全选</i>
                                            </label>
                                            <label :for="item.id+'update'" class="bk-form-checkbox bk-checkbox-small">
                                                <input type="checkbox"
                                                v-model="tempEventData['subscription_form'][item.id]"
                                                :value="item.id+'update'"
                                                :id="item.id+'update'"><i class="bk-checkbox-text" title="编辑">编辑</i>
                                            </label>
                                            <label for="moduletransfer" class="bk-form-checkbox bk-checkbox-small">
                                                <input type="checkbox"
                                                value="moduletransfer"
                                                v-model="tempEventData['subscription_form'][item.id]"
                                                id="moduletransfer"><i class="bk-checkbox-text" title="模块转移">模块转移</i>
                                            </label>
                                        </div>
                                    </template>
                                </template>
                                <template v-else>
                                    <label for="" class="label-name" :title="item.name">{{item.name}}</label>
                                    <div class="options">
                                        <label :for="item.id+'all'" class="bk-form-checkbox bk-checkbox-small">
                                            <input type="checkbox"
                                            :value="item.id+'all'"
                                            @change="checkAll(item.id)"
                                            :checked="tempEventData['subscription_form'][item.id].length == 3"
                                            :id="item.id+'all'"><i class="bk-checkbox-text" title="全选">全选</i>
                                        </label>
                                        <label :for="item.id+'create'" class="bk-form-checkbox bk-checkbox-small">
                                            <input type="checkbox"
                                            v-model="tempEventData['subscription_form'][item.id]"
                                            :value="item.id+'create'"
                                            :id="item.id+'create'"><i class="bk-checkbox-text" title="新增">新增</i>
                                        </label>
                                        <label :for="item.id+'update'" class="bk-form-checkbox bk-checkbox-small">
                                            <input type="checkbox"
                                            :value="item.id+'update'"
                                            v-model="tempEventData['subscription_form'][item.id]"
                                            :id="item.id+'update'"><i class="bk-checkbox-text" title="编辑">编辑</i>
                                        </label>
                                        <label :for="item.id+'delete'" class="bk-form-checkbox bk-checkbox-small">
                                            <input type="checkbox"
                                            :value="item.id+'delete'"
                                            v-model="tempEventData['subscription_form'][item.id]"
                                            :id="item.id+'delete'"><i class="bk-checkbox-text" title="删除">删除</i>
                                        </label>
                                    </div>
                                </template>
                            </li>
                        </ul>
                    </transition>
                </li>
            </ul>
        </div>
        <footer class="footer">
            <bk-button type="primary" class="btn" @click="save">保存</bk-button>
            <bk-button type="default" class="btn vice-btn" @click="cancel">取消</bk-button>
        </footer>
        <div class="pop-master" v-show="isPopShow">
            <v-pop
                :callbackURL="tempEventData['callback_url']"
                :isShow="isPopShow"
                @closePop="closePop"
            ></v-pop>
        </div>
    </div>
</template>

<script>
    import vPop from './pop'
    import {mapGetters} from 'vuex'
    export default {
        props: {
            curEvent: {
                default: {}
            },
            type: {             // 当前操作类型 编辑or新增  edit/add
                default: 'add'
            },
            isShow: {           // 弹窗显示状态
                default: false,
                type: Boolean
            }
        },
        data () {
            return {
                isPopShow: false,           // 测试推送弹窗
                eventPushList: [],          // 事件推送可选列表
                eventData: {                    // 订阅事件相关参数
                    subscription_name: '',   // 订阅名
                    system_name: '',         // 系统名称
                    callback_url: 'http://',        // 回调地址
                    confirm_mode: 'httpstatus',        // 回调成功确认模式   httpstatus/regular
                    confirm_pattern: '200',     // 回调成功标志
                    subscription_form: [],   // 订阅单
                    time_out: 60             // 超时时间  单位 秒
                },
                tempEventData: {                // 保存用户当前操作的内容 在保存成功后才赋值给eventData
                    subscription_name: '',   // 订阅名
                    system_name: '',         // 系统名称
                    callback_url: 'http://',        // 回调地址
                    confirm_mode: 'httpstatus',        // 回调成功确认模式   httpstatus/regular
                    // confirm_pattern: '200',     // 回调成功标志
                    confirm_pattern: {
                        httpstatus: '200',
                        regular: ''
                    },
                    subscription_form: {},   // 订阅单
                    time_out: 60             // 超时时间  单位 秒
                },
                subscriptionFormError: false
            }
        },
        computed: {
            ...mapGetters([
                'bkSupplierAccount',
                'allClassify'
            ]),
            /*
                推送事件已选数量
            */
            selectNum () {
                let num = 0
                for (let key in this.tempEventData['subscription_form']) {
                    let val = this.tempEventData['subscription_form'][key]
                    if (val.length) {
                        num += val.length
                    }
                }
                if (num) {
                    this.subscriptionFormError = false
                }
                return num
            }
        },
        watch: {
            isShow (val) {
                if (val) {
                    this.isPopShow = false
                    // 清空数据
                    this.clearData()
                    this.getEventPushList()
                    if (this.type === 'edit') {
                        let arr = this.curEvent['subscription_form']
                        let subscriptionForm = {}
                        arr.map(val => {
                            if (val === 'hostupdate') {
                                val = 'moduleupdate'
                            }
                            if (val === 'moduletransfer') {
                                if (subscriptionForm.hasOwnProperty('module')) {
                                    subscriptionForm['module'].push('moduletransfer')
                                } else {
                                    subscriptionForm['module'] = ['moduletransfer']
                                }
                            } else {
                                let key = val.substr(0, val.length - 6)
                                if (subscriptionForm.hasOwnProperty(key)) {
                                    subscriptionForm[key].push(val)
                                } else {
                                    subscriptionForm[key] = [val]
                                }
                            }
                        })
                        
                        this.tempEventData = {
                            subscription_id: this.curEvent['subscription_id'],
                            subscription_name: this.curEvent['subscription_name'],
                            system_name: this.curEvent['system_name'],
                            callback_url: this.curEvent['callback_url'],
                            confirm_mode: this.curEvent['confirm_mode'],
                            confirm_pattern: {
                                httpstatus: this.curEvent['confirm_mode'] === 'httpstatus' ? this.curEvent['confirm_pattern'] : '',
                                regular: this.curEvent['confirm_mode'] === 'regular' ? this.curEvent['confirm_pattern'] : ''
                            },
                            subscription_form: {...this.tempEventData['subscription_form'], ...subscriptionForm},
                            time_out: this.curEvent['time_out']
                        }
                        this.eventData = {...this.tempEventData}
                    }
                }
            }
        },
        methods: {
            /*
                全选按钮
            */
            checkAll (objId) {
                if (event.target.checked) {
                    if (objId === 'host') {
                        this.tempEventData['subscription_form'][objId] = ['hostcreate', 'hostdelete']
                    } else if (objId === 'module') {
                        this.tempEventData['subscription_form'][objId] = ['moduletransfer', 'moduleupdate']
                    } else {
                        this.tempEventData['subscription_form'][objId] = [`${objId}create`, `${objId}update`, `${objId}delete`]
                    }
                } else {
                    this.tempEventData['subscription_form'][objId] = []
                }
            },
            eventHeight (len) {
                return `height: ${len * 32}px`
            },
            toggleEventList (classify) {
                classify.isHidden = !classify.isHidden
            },
            testPush () {
                this.$validator.validate('http').then(res => {
                    if (res) {
                        this.isPopShow = true
                    }
                })
            },
            /*
                保存按钮
            */
            save () {
                this.checkParams().then(res => {
                    if (res) {
                        let url = ''
                        let method = ''
                        let appid = 0
                        if (this.type === 'add') {  // 新增
                            url = `event/subscribe/${this.bkSupplierAccount}/${appid}`
                            method = 'post'
                        } else { // 编辑
                            url = `event/subscribe/${this.bkSupplierAccount}/${appid}/${this.curEvent['subscription_id']}`
                            method = 'put'
                        }
                        let params = this.$deepClone(this.tempEventData)
                        params['confirm_pattern'] = this.tempEventData['confirm_mode'] === 'httpstatus' ? this.tempEventData['confirm_pattern']['httpstatus'] : this.tempEventData['confirm_pattern']['regular']
                        let subscriptionForm = ''
                        for (let key in params['subscription_form']) {
                            if (params['subscription_form'][key].length) {
                                if (key === 'module') {
                                    for (let i = 0; i < params['subscription_form'][key].length; i++) {
                                        if (params['subscription_form'][key][i] === 'moduleupdate') {
                                            params['subscription_form'][key][i] = 'hostupdate'
                                        }
                                    }
                                }
                                subscriptionForm += params['subscription_form'][key].join(',')
                                subscriptionForm += ','
                            }
                        }
                        subscriptionForm = subscriptionForm.substr(0, subscriptionForm.length - 1)
                        params['subscription_form'] = subscriptionForm
                        params['time_out'] = parseInt(params['time_out'])
                        this.$axios({
                            url: url,
                            method: method,
                            data: params
                        }).then(res => {
                            if (res.result) {
                                this.$alertMsg('保存成功', 'success')
                                this.eventData = {...this.tempEventData}
                                if (this.type === 'add') {
                                    this.$emit('saveSuccess', res.data['subscription_id'])
                                } else {
                                    this.$emit('saveSuccess')
                                }
                            } else {
                                this.$alertMsg(res['bk_error_msg'])
                            }
                        })
                    }
                })
            },
            /*
                获取推送事件列表
            */
            getEventPushList () {
                this.eventPushList = []
                let subscriptionForm = {}
                let eventPushList = []
                this.allClassify.map((classify, index) => {
                    // 去掉主机管理和业务拓扑
                    if (classify.hasOwnProperty('bk_objects') && classify['bk_objects'].length && classify['bk_classification_id'] !== 'bk_host_manage' && classify['bk_classification_id'] !== 'bk_biz_topo') {
                        let event = {
                            name: classify['bk_classification_name'],
                            isHidden: false,
                            children: []
                        }
                        classify['bk_objects'].map(val => {
                            event.children.push({
                                id: val['bk_obj_id'],
                                name: val['bk_obj_name']
                            })
                            subscriptionForm[val['bk_obj_id']] = []
                        })
                        eventPushList.push(event)
                    }
                })
                subscriptionForm['host'] = []
                subscriptionForm['module'] = []
                this.$set(this.tempEventData, 'subscription_form', subscriptionForm)
                eventPushList.unshift({
                    isDefault: true,
                    isHidden: false,
                    name: '主机业务',
                    children: [{
                        id: 'host',   // 该名字只是为了做区分
                        name: '资源池'
                    }, {
                        id: 'module',     // 该名字只是为了做区分
                        name: '主机'
                    }]
                })
                this.eventPushList = eventPushList
            },
            /*
                检查参数是否合法
            */
            async checkParams () {
                let result = false
                await this.$validator.validateAll().then(res => {
                    result = res
                })
                if (!result) {
                    return false
                }
                if (this.selectNum === 0) {
                    this.subscriptionFormError = true
                    return false
                }
                return true
            },
            /*
                清空数据
            */
            clearData () {
                this.tempEventData = {
                    subscription_name: '',
                    system_name: '',
                    callback_url: 'http://',
                    confirm_mode: 'httpstatus',
                    confirm_pattern: {
                        httpstatus: '200',
                        regular: ''
                    },
                    subscription_form: {},
                    time_out: 60
                }
                this.eventData = {...this.tempEventData}
            },
            closePop () {
                this.isPopShow = false
            },
            cancel () {
                this.$emit('cancel')
            }
        },
        components: {
            vPop
        }
    }
</script>

<style lang="scss" scoped>
    $primary-color: #737987;
    $danger-color: #ff3737;
    .slide-enter-active, .slide-leave-active{
        transition: height .3s;
        overflow: hidden;
    }
    .slide-enter, .slide-leave-to{
        height: 0 !important;
    }
    .detail-wrapper{
        position: relative;
        height: calc(100% - 60px);
        .text-danger{
            color: $danger-color;
        }
        .pop-master{
            position: absolute;
            left: 0;
            top: 0;
            bottom: 0;
            right: 0;
        }
        .detail-box{
            height: calc(100% - 63px);
            padding: 40px 40px 20px 20px;
            overflow-y: auto;
            &::-webkit-scrollbar{
                width: 6px;
                height: 5px;
            }
            &::-webkit-scrollbar-thumb{
                border-radius: 20px;
                background: #a5a5a5;
            }
        }
        color: $primary-color;
        .form-item{
            width: 100%;
            margin-bottom: 20px;
            &:after{
                display: block;
                content: "";
                clear: both;
            }
            .btn{
                display: inline-block;
                font-size: 14px;
                width: 96px;
                height: 36px;
                line-height: 36px;
                margin-right: 11px;
                background: #6b7baa;
                border-color: #6b7baa;
                color: #fff;
                text-decoration:none;
                border-radius: 2px;
                text-align: center;
                vertical-align: middle;
                &:hover{
                    background-color: #4d597d;
                    border-color: #4d597d;
                    opacity: 1;
                }
            }
            .color-danger{
                border-color: $danger-color;
                color: $danger-color;
            }
            .label-name{
                position: relative;
                float: left;
                width: 110px;
                text-align: right;
                line-height: 36px;
                font-size: 14px;
                .color-danger{
                    position: absolute;
                    top: 2px;
                    right: -6px;
                    color: $danger-color;
                }
            }
            .item-content{
                margin-left: 22px;
                width: 420px;
                float: left;
                .bk-form-radio{
                    cursor: pointer;
                    height: 32px;
                    margin-bottom: 10px;
                    color: $primary-color;
                    input[type="radio"]{
                        margin-top: -2px;
                    }
                }
                .bk-form-input.number{
                    display: block;
                    width: 97px;
                }
                &.length-short{
                    position: relative;
                    input{
                        width: 97px;
                        margin-right: 10px;
                    }
                    .unit{
                        position: absolute;
                        left: 110px;
                        top: 8px;
                    }
                }
            }
        }
        .info{
            background: #f9f9f9;
            width: 100%;
            padding-left: 20px;
            height: 44px;
            line-height: 44px;
            .num{
                color: #479cff;
                font-weight: bold;
            }
        }
        .event-wrapper{
            .event-box{
                padding: 13px 20px;
                border-bottom: 1px solid #eceef5;
                .event-title{
                    margin-right: 10px;
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                    height: 32px;
                    line-height: 32px;
                }
                .bk-icon{
                    float: right;
                    line-height: 32px;
                    width: 32px;
                    text-align: center;
                    margin-right: -6px;
                    transition: all .5s;
                    cursor: pointer;
                    &.up{
                        transform: rotate(180deg);
                    }
                }
                ul{
                    width: 680px;
                    float: left;
                }
            }
            .event-item{
                padding: 7px 0;
                height: 32px;
                line-height: 18px;
                &:after{
                    display: block;
                    content: "";
                    clear: both;
                }
                .label-name{
                    float: left;
                    width: 158px;
                    text-align: right;
                    font-size: 14px;
                    margin-right: 22px;
                    font-weight: bold;
                    white-space: nowrap;
                    text-overflow: ellipsis;
                    overflow: hidden;
                }
                .options{
                    label{
                        height: 18px;
                        width: 110px;
                        margin-right: 10px;
                        &.bk-form-checkbox{
                            padding: 0;
                            cursor: pointer;
                        }
                        input[type="checkbox"]{
                            margin-right: 10px;
                        }
                        .bk-checkbox-text{
                            display: inline-block;
                            width: 82px;
                            overflow: hidden;
                            text-overflow: ellipsis;
                            white-space: nowrap;
                            color: $primary-color;
                        }
                    }
                    &.wd{
                        .bk-form-checkbox{
                            width: auto;
                            .bk-checkbox-text{
                                width: auto;
                            }
                        }
                    }
                }
            }
        }
        .footer{
            height: 63px;
            line-height: 63px;
            background: #f9f9f9;
            font-size: 0;
            padding-left: 20px;
            .btn{
                font-size: 14px;
                width: 110px;
                height: 34px;
                line-height: 32px;
                margin-right: 11px;
            }
            .cancel:hover{
                color: $primary-color;
                border: 1px solid $primary-color;
            }
        }
    }
</style>
