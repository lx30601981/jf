<template>
    <div>
        <!--操作按钮行，搜索框-->
        <div style="margin-bottom:6px;">
            <!--添加-->
            <Button type="primary" icon="plus" @click="showModal('add')">{{$t('common.add')}}</Button>
            <Input v-model="grid.queryParams.queryParams" :placeholder="$t('role.queryTip')" style="width: 360px;float:right">
                <Button slot="append" icon="ios-search"  @click="loadTable(1)"></Button>
            </Input>
        </div>
        <!--Table列表-->
        <Table height="400" border stripe size="small" 
            :columns="grid.columns" :data="grid.result" 
            :no-data-text="grid.noDataMessage">
        </Table>
        <!--分页-->
        <div style="margin: 5px;overflow: hidden">
            <div style="float:right">
                <Page @on-change="onChange"
                    @on-page-size-change="onPageSizeChange"
                    :total="grid.page.total" 
                    :page-size="grid.page.pageSize" 
                    :current="grid.page.curPage" 
                    size="small"
                    placement="top" show-elevator show-sizer show-total>
                </Page>
            </div>
        </div>
        <!--删除确认Modal-->
        <Modal v-model="grid.deleteModal" width="360">
            <p slot="header" style="color:#f60;text-align:center">
                <Icon type="information-circled"></Icon>
                <span>{{$t('common.deleteConfirm')}}</span>
            </p>
            <div style="text-align:center">
                <p>{{$t('common.deleteConfirmMsg')}}</p>
            </div>
            <div slot="footer">
                <Button type="error" size="large" long :loading="grid.deleteLoading" @click="deleteRecord">{{$t('common.delete')}}</Button>
            </div>
        </Modal>
        <!--添加编辑Modal-->
        <Modal v-model="grid.addOrEditModal" width="760" 
            :title="$t('common.addEdit')"
        >
            <Form ref="formMain" :model="grid.formValidate" :rules="grid.ruleValidate" :label-width="110">
                <Row>
                    <Col span="12">
                        <Form-item :label="$t('role.code')" prop="roleCode">
                            <Input v-model="grid.formValidate.roleCode"></Input>
                        </Form-item>
                    </Col>
                    <Col span="12">
                        <Form-item :label="$t('role.name')" prop="roleName">
                           <Input v-model="grid.formValidate.roleName"></Input>
                        </Form-item>
                    </Col>
                </Row>
                <Row>
                    <Col span="12">
                        <Form-item :label="$t('common.applyStatus')" prop="applyStatus">
                            <Radio-group v-model="grid.formValidate.applyStatus">
                                <Radio label="Y">{{$t('common.yes')}}</Radio>
                                <Radio label="N">{{$t('common.no')}}</Radio>
                            </Radio-group>
                        </Form-item>
                    </Col>
                    <Col span="12">
                        <Form-item :label="$t('common.owner')" prop="roleOwner">
                            <Input v-model="grid.formValidate.roleOwner"></Input>
                        </Form-item>
                    </Col>
                </Row>
                <Form-item :label="$t('common.desc')" prop="roleDesc">
                    <Input v-model="grid.formValidate.roleDesc" type="textarea" :autosize="{minRows: 2,maxRows: 5}"></Input>
                </Form-item>
            </Form>
            <div slot="footer">
                <Button type="text"  @click="grid.addOrEditModal=false">{{$t('common.cancel')}}</Button>
                <Button type="primary" :loading="grid.saveLoading" @click="saveRecord">{{$t('common.save')}}</Button>
            </div>
        </Modal>
    </div>
</template>
<script>
    import Util from '../../../../libs/util';
    export default {
        data(){
            let vue=this;
            return {
                grid:{
                    //列配置信息
                    columns:[
                        {title: vue.$t('role.code'),key: 'roleCode',width:120},
                        {title: vue.$t('role.name'),key: 'roleName',width:250},
                        {title: vue.$t('common.applyStatus'),key: 'applyStatus',align: 'center',width:120,render:(h,params)=>{
                            return Util.statusRender(h,params.row.applyStatus,params.row);
                        }},
                       
                        {title: vue.$t('common.owner'),key: 'roleOwnerName',width:150},
                        {title: vue.$t('common.desc'),key: 'roleDesc',width:400},
                        {title: vue.$t('common.lastUpdatedBy'),key: 'updateUserAccount',width:150},
                        {title: vue.$t('common.lastUpdationDate'),key: 'lastUpdationDate',width:160,render:(h,params)=>{
                            return Util.toDateStr(params.row.lastUpdationDate);
                        }},
                        {title: vue.$t('common.operate'),key: 'action', width:300, align: 'center',fixed: 'right',render: (h, params) => {
                                return h('div', [
                                    h('Button', {
                                        props: {type: 'primary',size: 'small'},
                                        style: {marginRight: '5px'},
                                        on: {click: () => {
                                            this.showModal('edit',params.row.roleId);
                                        }}
                                    }, this.$t('common.edit')),
                                    h('Button', {
                                        props: {type: 'error',size: 'small'},
                                        style: {marginRight: '5px'},
                                        on: {click: () => {
                                            vue.grid.deleteModal=true;
                                            this.deleteKey=params.row.roleId;
                                        }}
                                    }, this.$t('common.delete')),
                                    h('Button', {
                                        props: {type: 'primary',size: 'small'},
                                        style: {marginRight: '5px'},
                                        on: {click: () => {
                                           
                                        }}
                                    }, this.$t('role.grant')),
                                    h('Button', {
                                        props: {type: 'primary',size: 'small'},
                                        style: {marginRight: '5px'},
                                        on: {click: () => {
                                           
                                        }}
                                    }, this.$t('role.users')),
                                    h('Button', {
                                        props: {type: 'primary',size: 'small'},
                                        on: {click: () => {
                                           
                                        }}
                                    }, this.$t('role.groups'))
                                ]);
                            }
                        }
                    ],
                    //列表数据
                    result:[],
                    //分页信息
                    page:{
                        total:0,
                        curPage:1,
                        pageSize:10
                    },
                    //查询参数
                    queryParams:{},
                    //是否显示删除确认框
                    deleteModal:false,
                    //是否正在删除中
                    deleteLoading:false,
                    //是否显示添加编辑框
                    addOrEditModal:false,
                    //是否正在保存中
                    saveLoading:false,
                    formValidate: {
                        applyStatus: 'N'
                    },
                    ruleValidate: {
                        roleCode: [
                            { required: true, message: vue.$t('common.notNull'), trigger: 'blur' }
                        ],
                        roleName: [
                            { required: true, message: vue.$t('common.notNull'), trigger: 'blur' }
                        ],
                        applyStatus: [
                            { required: true, message: vue.$t('common.notNull'), trigger: 'change' }
                        ],
                        roleOwner: [
                            { required: true, message: vue.$t('common.notNull'), trigger: 'change' }
                        ]
                    }
                }
            }
        },
        methods:{
            /**
             * 加载数据
             */
            loadTable:function(curPage=1){
                let vue=this;
                vue.grid.page.curPage=curPage;
                Util.loadGrid(vue,"grid","services/jf/security/role/find/page");
            },
            /**
             * 显示添加&编辑Modal
             */
            showModal:function(type,roleId){
                let vue=this;
                Util.showAddOrEditModal(vue,'grid','formMain',type,'services/jf/security/role/find/single',{roleId:roleId});
            },
            /**
             * 删除数据
             */
            deleteRecord:function(){
                let vue=this;
                Util.deleteGrid(vue,'grid','services/jf/security/role/delete',{roleId:vue.deleteKey},function(){
                    vue.loadTable();
                });
            },
            /**
             * 保存记录
             */
            saveRecord:function(){
                let vue=this;
                Util.saveGrid(vue,'grid','formMain',
                    'services/jf/security/role/create',
                    'services/jf/security/role/update','roleId',function(){
                        vue.loadTable();
                    }
                );
            },
            /**
             * 翻页
             */
            onChange:function(curPage){
                this.loadTable(curPage);
            },
            /**
             * 每页大小切换
             */
            onPageSizeChange:function(pageSize){
                let vue=this;
                vue.grid.page.pageSize=pageSize;
                vue.loadTable();
            }
        },
        created(){
            this.loadTable();
        }
    };
</script>