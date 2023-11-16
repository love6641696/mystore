> 前端html模板

```html
<!DOCTYPE html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org" >
<head>
    <th:block th:include="include :: vue3-css('修改订单管理')"/>
    <style>
    </style>
</head>
<body class="white-bg" id="app">
<div class="myshade myshadecolor"><img class="rotate" th:src="@{/img/loading.png?v=230422}"></div>
<el-container>
<el-main>
    <z-buttons  @submit="grobalEvent.submitHandler" perms="so:so_order3:" :dataid="grobalVar.form1.orderid"></z-buttons>
    <!--主子表-->
    <el-collapse v-model="grobalVar.activeName" @change="grobalEvent.setHeight(this.$refs.grid1)" style="height: 100%">
        <el-collapse-item name="master" ref="master">
            <template #title>
                <h4 style="margin-right: 5px">订单管理主表信息</h4><el-icon  @click.stop="this.zconfig.show(this.$refs.form1,{formid:'soOrder3_edit_form1'})" title="表单设置"><setting/></el-icon>
            </template>
            <el-form ref="form1" :model="grobalVar.form1" label-width="120px" :disabled="addi==3">
                <el-row :gutter="10">
                    <el-col v-if="this.XEUtils.i18n.findV('soOrder3_edit_form1','orderid')" :span="this.XEUtils.i18n.findW('soOrder3_edit_form1','orderid')" :style="{order:this.XEUtils.i18n.findI('soOrder3_edit_form1','orderid')}">
                        <el-form-item prop="orderid" :label="this.XEUtils.i18n.findT('soOrder3_edit_form1','orderid','订单ID（平台唯一编号）')" :label-width="this.XEUtils.i18n.findLW('soOrder3_edit_form1','orderid')" :required="this.XEUtils.i18n.findQ('soOrder3_edit_form1','orderid')">
                            <el-input v-model="grobalVar.form1.orderid" :disabled="this.XEUtils.i18n.findR('soOrder3_edit_form1','orderid')"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </el-collapse-item>
        <el-collapse-item  name="detail">
            <template #title>
                <h4 style="margin-right: 5px">订单管理明细信息</h4><el-icon> <Grid /></el-icon>
            </template>
            <vxe-grid v-bind="gridOptions1" ref="grid1">
                <template #toolbar_buttons="{$grid}">
                    <vxe-button icon="vxe-icon-copy" content="复制"  @click="grobalEvent.copy($grid)"></vxe-button>
                    <vxe-button icon="vxe-icon-delete" content="删除"  @click="$grid.removeCheckboxRow()"></vxe-button>
                </template>
                <template #toolbar_tools="{$grid}">
                    <vxe-button @click="this.zconfig.show($grid)" icon="vxe-icon-setting" circle style="margin-right: 10px"></vxe-button>
                </template>
            </vxe-grid>
        </el-collapse-item>
    </el-collapse>
</el-main>
</el-container>
```
> 前端js模板

```javascript
<th:block th:include="include :: vue3-js"/>
<script th:inline="javascript">
    const prefix = ctx + 'so/so_order3'

    $.vxeTable.registry({
        setup(){
            //统一编码风格，按照以下模板编写业务代码
            const {proxy} = getCurrentInstance();
            window.proxy = proxy;
            let $grid1=null
            const addi=ref([[${addi}]] || 0)  //1:新增 2:编辑 3:查看

            //响应式变量定义
            const grobalVar= reactive({
                activeName:['master','detail'],
                form1:[[${soOrder3}]] || {},
            })

            //变量定义
            const grobalVar2= {
            }

            //事件定义
            const grobalEvent={
                //折叠栏变化后重置表格高度
                async setHeight($grid){
                    proxy.XEUtils.sleep(100*3)
                    const h=document.body.clientHeight-proxy.$refs.master.$el.clientHeight-80
                    $grid.setHeight(h)
                },
                //复制行
                copy($grid){
                    const data = $grid.getCheckboxRecords(true);
                    if (data.length == 0) {
                        $grid.addRows()
                        return
                    }
                    const data0=data[0]
                    const row=proxy.XEUtils.assign({},data0, {'_X_ROW_KEY': ''}) //注意：此处还需清空主键id
                    const index=$grid.getRowSeq(data0)-1
                    $grid.addRows(row,index)
                },
                //保存
                async submitHandler(){
                    await proxy.$refs.form1.validate() //如果需要弹出错误提示，可在try catch中 proxy.modal.error(proxy.XEUtils.getError(errMap))
                    const errMap = await $grid1.validate(true)
                    if(errMap){
                        proxy.modal.error(proxy.XEUtils.getError(errMap))
                        return false;
                    }
                    const id=grobalVar.form1.orderid
                    grobalVar.form1.isconvert=true
                    grobalVar.form1.mesFactorderieList=$grid1.getTableData().tableData
                    const result=await proxy.operate.save({url:prefix+ (id?'/edit':'/add'),data:JSON.stringify(grobalVar.form1)});
                    if(result.code!=0){
                        proxy.modal.error(result.msg)
                        return;
                    }
                    proxy.modal.success(result.msg)
                    const data=result.data
                    grobalVar.form1=data
                    proxy.XEUtils.mitt.emit(prefix)
                    const tabledata=data.mesFactorderieList
                    $grid1.reloadData(tabledata)
                }
            }

            //表格参数定义
            const gridOptions1 = reactive($.vxeTable.init({
                formid:'soOrder3_edit',
                id:'grid1',
                type:'edit',
                height:500,
                addi:addi.value,
                data:grobalVar.form1?.mesFactorderieList || [],
                columnConfig:{
                    width:120
                },
                toolbarConfig: {
                    zoom:true,
                    slots: {
                        buttons: 'toolbar_buttons',
                        tools:'toolbar_tools',
                    }
                },
                editRules:{
                },
                columns:[
                    {
                        field: 'xxx1',
                        title: 'input编辑框渲染器',
                        index: 10,
                        editRender: {name:'$input',props:{clearable:true}},
                    },
                    {
                        field: 'xxx2',
                        title: '带搜索按钮的input编辑框渲染器',
                        index: 20,
                        editRender: {name:'zinput',props:{clearable:true}},
                    },
                    {
                        field: 'xxx3',
                        title: '下拉框渲染器',
                        index: 30,
                        editRender: {name:'zselect',props: {options:grobalVar2.getways}},
                    },
                    {
                        field: 'xxx4',
                        title: '开关组件渲染器',
                        index: 40,
                        editRender: {name:'zswitch',props:{clearable:true}},
                    },
                    {
                        field: 'xxx5',
                        title: '时间组件渲染器',
                        index: 50,
                        editRender: {name:'zdate'},
                    },
                    {
                        field: 'xxx6',
                        title: '图片组件渲染器',
                        index: 60,
                        editRender: {name:'zimage',props:{clearable:true}},
                    },
                ]
            }))

            onMounted(()=>{
                $('.myshade').remove()
                $grid1=proxy.$refs.grid1
                proxy.XEUtils.disabled(addi.value)
            })

            //对外暴露属性和方法
            return {
                addi,
                gridOptions1,
                grobalVar,
                grobalVar2,
                grobalEvent
            }
        }
    })
</script>
</body>
</html>
```
