<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getProducts">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleViewProperties">显示属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleSkuProperties">SKU属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">上架</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">下架</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="标题" width="250" sortable>
            </el-table-column>
            <el-table-column prop="subName" label="副标题" width="150" sortable>
            </el-table-column>
            <!--隐藏的属性-->
            <el-table-column v-if="false" prop="medias" label="媒体资源"  sortable>
            </el-table-column>
            <el-table-column v-if="false" prop="content" label="媒体资源"  sortable>
            </el-table-column>
            <el-table-column v-if="false" prop="description" label="媒体资源"  sortable>
            </el-table-column>
            <!--隐藏-->

            <el-table-column prop="pt.name" label="商品类型" width="100" sortable>
            </el-table-column>
            <el-table-column prop="brand.name" label="商品品牌" width="100" sortable>
            </el-table-column>
            <el-table-column prop="onSaleTime" label="上架时间" width="150" :formatter="formatterOnSaleTime" sortable>
            </el-table-column>
            <el-table-column prop="offSaleTime" label="下架时间" width="150" :formatter="formatterOffSaleTime" sortable>
            </el-table-column>
            <el-table-column prop="state" label="状态" min-width="50" sortable>
                <template scope="scope">
                    <span v-if="scope.row.state==0" style="color:red">下架</span>
                    <span v-else style="color:green">上架</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="size" :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--编辑界面-->
        <el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="标题" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="副标题" prop="subName">
                    <el-input v-model="editForm.subName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类型">
                    <el-cascader style="width:100%"
                                 :options="productTypes"
                                 v-model="editForm.productType"
                                 :props="productProps">
                    </el-cascader>
                </el-form-item>
                <el-form-item label="品牌">
                    <el-input v-model="editForm.brandId"></el-input>
                </el-form-item>
                <el-form-item label="媒体属性">
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:9527/services/common/file/upload"
                            :file-list="mediaList"
                            :on-success="handleUploadSeccess"
                            list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="商品描述">
                    <el-input v-model="editForm.description"></el-input>
                </el-form-item>
                <el-form-item label="商品详情">
                    <!--<quill-editor
                            v-model="addForm.content"
                            ref="myQuillEditor"
                            :options="editorOption">
                    </quill-editor>-->
                    <!--富文本 编辑器 传文件只传地址 导入了@/components/SquillEditorFastdfs.vue-->
                    <SquillEditorFastdfs host="http://172.20.10.12" v-model="editForm.content" uploadUrl='http://localhost:9527/services/common/file/upload'></SquillEditorFastdfs>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--新增界面-->
        <el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
                <el-form-item label="标题" prop="name">
                    <el-input v-model="addForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="副标题" prop="subName">
                    <el-input v-model="addForm.subName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类型" prop="productType">
                    <el-cascader style="width:100%"
                                 :options="productTypes"
                                 v-model="addForm.productType"
                                 :props="productProps">
                    </el-cascader>
                </el-form-item>
                <el-form-item label="品牌"prop="brandId">
                    <el-input v-model="addForm.brandId"></el-input>
                </el-form-item>
                <el-form-item label="媒体属性">
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:9527/services/common/file/upload"
                            :file-list="mediaList"
                            :on-success="handleUploadSeccess"
                            list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="商品描述">
                    <el-input v-model="addForm.description"></el-input>
                </el-form-item>
                <el-form-item label="商品详情">
                    <!--<quill-editor
                            v-model="addForm.content"
                            ref="myQuillEditor"
                            :options="editorOption">
                    </quill-editor>-->
                    <!--富文本 编辑器 传文件只传地址 导入了@/components/SquillEditorFastdfs.vue-->
                    <SquillEditorFastdfs host="http://172.20.10.12" v-model="addForm.content" uploadUrl='http://localhost:9527/services/common/file/upload'></SquillEditorFastdfs>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
        <!-------------------------------------------显示属性维护的模态框-------------------------------------->
        <el-dialog
                title="显示属性管理"
                :visible.sync="viewDialogVisible"
                width="40%">

            <!--卡片-->
            <el-card class="box-card">
                <div v-for="index in viewProperties.length" :key="index" class="text item" style="margin-bottom: 5px">
                    <el-row>
                        <el-col :span="3">{{viewProperties[index-1].specName}}:</el-col>
                        <el-col :span="21">
                            <el-input v-model="viewProperties[index-1].value"></el-input>
                        </el-col>
                    </el-row>
                </div>
            </el-card>


            <span slot="footer" class="dialog-footer">
                <el-button @click="viewDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="subViewProperties">确 定</el-button>
          </span>
        </el-dialog>
        <!-------------------------------------------sku属性维护的模态框-------------------------------------->

        <el-dialog title="SKU属性管理" :visible.sync="skuDialogVisible" width="60%">
            <!--卡片     外层的卡片代表者一个sku属性-->
            <el-card class="box-card" v-for="skuProperty in skuProperties" style="margin-bottom: 5px">
                <!--内层的div代表sku属性的选项-->
                <div slot="header" class="clearfix">
                    <span>{{skuProperty.specName}}</span>
                </div>
                <!--length+1是为了每次写最后一个输入框的时候就增加一层-->
                <div v-for="index in skuProperty.options.length+1" :key="index" class="text item">
                    <el-input v-model="skuProperty.options[index-1]" style="width:80%"></el-input>
                    <!--删除一条-->
                    <el-button icon="el-icon-delete" @click="skuProperty.options.splice(index-1,1)" style="width:10%"></el-button>
                </div>
            </el-card>
            <el-table :data="skus" border style="width: 100%">
                <el-table-column type="index" width="50"></el-table-column>
                <!--确定列名称-->
                <template v-for="(value,key) in skus[0]">
                    <!--更改价格和库存单元格和标题-->
                    <el-table-column v-if="key=='price'" :prop="key" label="价格">
                        <template scope="scope">
                            <el-input v-model="scope.row.price"></el-input>
                        </template>
                    </el-table-column>
                    <el-table-column v-else-if="key=='availableStock'" :prop="key" label="库存">
                        <template scope="scope">
                            <el-input v-model="scope.row.availableStock"></el-input>
                        </template>
                    </el-table-column>
                    <!--隐藏sku_index属性-->
                    <el-table-column v-else-if="key!='sku_index'" :prop="key" :label="key">
                    </el-table-column>
                </template>
            </el-table>
            <span slot="footer" class="dialog-footer">
                <el-button @click="skuDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="subSkuProperties">确 定</el-button>
          </span>
        </el-dialog>

    </section>
</template>


<script>
    import SquillEditorFastdfs from '@/components/SquillEditorFastdfs.vue';
    export default {
        components:{
            SquillEditorFastdfs //富文本框上传组件
        },
        data() {
            return {
                //sku模态框
                skuDialogVisible:false,
                skuProperties:[],
                skus:[],//sku所有组合
                //显示属性模态框
                viewDialogVisible:false,
                viewProperties:[],//显示属性 通过选中的行的 productId查出来
                editorOption:{},
                //商品类型
                productTypes: [],
                //级联选择器的属性配置
                productProps: {
                    label: "name",
                    value: "id"
                },
                mediaList:[],
                filters: {
                    keyword: ''
                },
                products: [],
                total: 0,
                page: 1,
                size:10,
                listLoading: false,
                sels: [],//列表选中列

                editFormVisible: false,//编辑界面是否显示
                editLoading: false,
                editFormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ]
                },
                //编辑界面数据
                editForm: {
                    id:"",
                    name:'',
                    subName:'',
                    productType:null,
                    brandId:null,
                    maxPrice:null,
                    minPrice:null,
                    description:'',
                    content:'',
                    medias:"",
                    mediasArr:[]
                },

                addFormVisible: false,//新增界面是否显示
                addLoading: false,
                addFormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ]
                },
                //新增界面数据
                addForm: {
                    name:'',
                    subName:'',
                    productType:null,
                    brandId:null,
                    maxPrice:null,
                    minPrice:null,
                    description:'',
                    content:'',
                    mediasArr:[]
                }

            }
        },
        methods: {

            //显示sku属性框
            handleSkuProperties(){
                //判断是否选中一行
                if(this.sels.length!=1) {
                    this.$message({
                        message: '只能选中一行',
                        type: 'warning'
                    });
                    return;
                }
                    //发送请求查询sku属性
                    let productId = this.sels[0].id;
                    this.$http.get("/product/product/skuProperties?productId="+productId)
                        .then((res)=>{
                            this.skuProperties=res.data;//返回的是数组（value，key），循环取value
                        });
                    //对sku的价格和库存进行回填
                    this.$http.get("/product/product/skus?productId="+productId)
                        .then(res=>{
                           let data =res.data;
                           this.skus.forEach(e1=>{
                               data.forEach(e2=>{
                                   if(e1.sku_index==e2.skuIndex){
                                       e1.price=e2.price;
                                       e1.availableStock=e2.availableStock;
                                   }
                               })
                           })
                        });
                //只选中一行的情况才会执行以下代码
                this.skuDialogVisible = true;
            },
            //确定提交sku
            subSkuProperties(){
                //准备请求参数
                let para = {}
                para.skuProperties = this.skuProperties;//保存到商品表
                para.productId = this.sels[0].id;
                para.skus = this.skus;//添加到sku表中
                //发送请求
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/product/product/skuProperties",para).then(res=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                message: '保存成功!',
                                type: 'success'
                            });
                            //关闭模态框
                            this.skuDialogVisible = false;
                        }else{
                            this.$message({
                                message: data.message,
                                type: 'error'
                            });
                        }
                    })
                })
            },
            //显示 显示属性模态框 handleViewProperties
            handleViewProperties(){
              //判断是否选中一行
              if(this.sels.length!=1){
                  this.$message({
                      message: '只能选中一行',
                      type: 'warning'
                  });
                  return;
              }//只选中一行的情况才会执行以下代码
              //发送请求查询ViewProperties 根据productId 查询从选中行中可以找到id
                let productId = this.sels[0].id;
                this.$http.get("/product/product/viewProperties?productId="+productId)
                    .then((res)=>{
                        this.viewProperties=res.data;//返回的是数组（value，key），循环取value
                    });
                //打开模态框
                this.viewDialogVisible = true;
            },//---------------------------------------------------------------------------

            //点击提交显示属性按钮
            subViewProperties(){
              //准备请求参数
                let para={};
                //存入t_product表中，要查product 需要productId，然后存入viewProperties
                para.productId=this.sels[0].id;
                para.viewProperties=this.viewProperties;
                //发送post请求
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/product/product/viewProperties",para)
                        .then(res=>{
                            let data=res.data;
                            if(data.success){
                                this.$message({
                                    message: '保存成功!',
                                    type: 'success'
                                });
                                //关闭模态框
                                this.viewDialogVisible = false;
                            } else {
                                this.$message({
                                    message: '保存失败!'+data.message,
                                    type: 'error'
                                });
                            }
                        });
                })

            },

            //-------------------------------------------------------------------------------


            //文件上传成功钩子函数
            handleUploadSeccess(response){
                this.addForm.mediasArr.push(response.data)
            },
            //加载商品类型
            getProductTypes() {
                this.$http.get("/product/productType/tree").then((res) => {
                    this.productTypes = this.getTreeData(res.data);
                })
            },
            getTreeData(data) {
                // 循环遍历json数据
                for (var i = 0; i < data.length; i++) {

                    if (data[i].children.length < 1) {
                        // children若为空数组，则将children设为undefined
                        data[i].children = undefined;
                    } else {
                        // children若不为空数组，则继续 递归调用 本方法
                        this.getTreeData(data[i].children);
                    }
                }
                return data;
            },
            //上架时间格式化
            formatterOnSaleTime:function(row, column){
                return row.onSaleTime?this.formatDate(row.onSaleTime):""
            },
            //下架时间格式化
            formatterOffSaleTime:function(row, column){
                return row.offSaleTime?this.formatDate(row.offSaleTime):""
            },
            //时间显示转换
            formatDate(longTime){
                let date = new Date(longTime);
                let year = date.getFullYear();//2019
                let month = date.getMonth()+1;//月份从0开始
                let day = date.getDate();//日
                let hour = date.getHours();
                let minute = date.getMinutes();
                let second = date.getSeconds();
                return year+"-"+this.numberFormatter(month)+"-"+this.numberFormatter(day)+" "
                    +this.numberFormatter(hour)+":"+this.numberFormatter(minute)+":"+this.numberFormatter(second);
            },
            numberFormatter(num){
                if(num<10){
                    return "0"+num;
                }
                return num;
            },
            //性别显示转换
            formatSex: function (row, column) {
                return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getProducts();
            },
            //获取商品列表
            getProducts() {
                let para = {
                    page: this.page,
                    size:this.size,
                    keyword: this.filters.keyword
                };
                this.listLoading = true;
                this.$http.post("/product/product/page",para).then(res=>{
                    let data = res.data;
                    this.total = data.total;
                    this.products = data.rows;
                    this.listLoading = false;
                });
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    //let para = { id: row.id };
                    this.$http.delete("/product/product/",{data: row.id})
                        .then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                            if(data.success){
                                this.$message({
                                    message: '删除成功!',
                                    type: 'success'
                                });
                                //关闭模态框
                                this.viewDialogVisible = false;
                            } else {
                                this.$message({
                                    message: '删除失败!'+data.message,
                                    type: 'error'
                                });
                            }
                    });
                    this.getProducts();
                })
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                this.mediaList=[];
                this.editForm= {
                    name:'',
                    subName:'',
                    productType:null,
                    brandId:null,
                    maxPrice:null,
                    minPrice:null,
                    description:'',
                    content:'',
                    mediasArr:[]
                };
                this.editForm = Object.assign({}, row);
                //跟新时间
                //this.editForm.updateTime=Date.now();
                // http://172.20.10.12/group1/M00/00/01/rBQKDFzicwOAT2X8ABDFfNGid1M175_big.png
                //当有logo的时候显示出来
                if(this.editForm.medias){
                        //todo 回填数据图片
                        let mediasArr =this.editForm.medias.split(',');
                        for(let i=0;i<mediasArr.length;i++){
                            this.mediaList.push({name: this.editForm.name,url:"http://172.20.10.12"+mediasArr[i]})
                            // console.log("mediasArr[i]============"+"http://172.20.10.12"+mediasArr[i])
                        }
                    console.log("this.mediaList============"+this.mediaList)
                    // this.mediaList = [{name: this.editForm.name,url:"http://172.20.10.12"+this.editForm.logo}];
                }
                //回填商品类型
                this.editForm.productType=this.changeDetSelect(this.editForm.productType,this.productTypes) //数据双向绑定
                //todo 回填商品描述和商品详情，这2条信息在t_product_ext中
                console.log("editForm.content====="+this.editForm.content);
            },
            changeDetSelect(key,treeData){
                let arr = []; // 在递归时操作的数组
                let returnArr = []; // 存放结果的数组
                let depth = 0; // 定义全局层级
                // 定义递归函数
                function childrenEach(childrenData, depthN) {
                    for (var j = 0; j < childrenData.length; j++) {
                        depth = depthN; // 将执行的层级赋值 到 全局层级
                        arr[depthN] = (childrenData[j].id);
                        if (childrenData[j].id == key) {
                            returnArr = arr.slice(0, depthN+1); //将目前匹配的数组，截断并保存到结果数组，
                            break
                        } else {
                            if (childrenData[j].children) {
                                depth ++;
                                childrenEach(childrenData[j].children, depth);
                            }
                        }
                    }
                    return returnArr;
                }
                return childrenEach(treeData, depth);
            },
            //显示新增界面
            handleAdd: function () {
                this.addFormVisible = true;
                this.mediaList=[];
                this.addForm= {
                    name:'',
                        subName:'',
                        productType:null,
                        brandId:null,
                        maxPrice:null,
                        minPrice:null,
                        description:'',
                        content:'',
                        mediasArr:[]
                }
            },
            //编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.editForm);
                            //medias
                            para.medias = this.arrToString(this.editForm.mediasArr);

                            para.productType = this.editForm.productType[this.editForm.productType.length-1];
                            //发送请求
                            this.$http.post("/product/product",para).then(res=>{
                                this.editLoading = false;
                                let data  = res.data;
                                if(data.success){
                                    this.$message({
                                        message: '保存成功',
                                        type: 'success'
                                    });
                                    this.$refs['editForm'].resetFields();
                                    this.editFormVisible = false;
                                    this.getProducts();
                                }else{
                                    this.$message({
                                        message: '保存失败',
                                        type: 'error'
                                    });
                                }

                            })

                        });
                    }
                });
            },
            //新增
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.addForm);
                            //medias
                            //para.medias = this.arrToString(this.addForm.mediasArr);
                            para.medias=this.addForm.mediasArr.join(',');
                            console.log("para.medias==="+para.medias)
                            para.productType = this.addForm.productType[this.addForm.productType.length-1];
                            //发送请求
                            this.$http.post("/product/product",para).then(res=>{
                                this.addLoading = false;
                                let data  = res.data;
                                if(data.success){
                                    this.$message({
                                        message: '保存成功',
                                        type: 'success'
                                    });
                                    this.$refs['addForm'].resetFields();
                                    this.addFormVisible = false;
                                    this.getProducts();
                                }else{
                                    this.$message({
                                        message: '保存失败',
                                        type: 'error'
                                    });
                                }

                            })

                        });
                    }
                });
            },
            arrToString(arr){
              let result = '[';
              for(let index=0;index<arr.length;index++){
                  result += "\""+arr[index]+"\""
                  if(index!=arr.length-1){
                      result+=","
                  }
              }
              result += ']';
              return result;
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = { ids: ids };
                    this.$http.delete("product/product",{data: ids})
                        .then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getProducts();
                    });
                }).catch(() => {

                });
            }
        },
        mounted() {
            this.getProducts();
            this.getProductTypes();
        },
        watch:{
            //监听属性的变化
            skuProperties:{
                handler(val, oldVal){
                    //动态生成skus值
                    let res = this.skuProperties.reduce((pre,cur)=>{
                        //准备一个数据来接收
                        let result=[];
                        //2层循环拼接
                        pre.forEach(e1=>{
                            //sku_index拼接 外层循环决定sku_index的个数，内层决定值
                            e1.sku_index = (e1.sku_index||'')+"_";
                            for(let i=0;i<cur.options.length;i++){
                                //当前的options
                                let e2 =cur.options[i];
                                //准备一个临时数组 接收上一次的结果
                                let temp={}
                                Object.assign(temp,e1);
                                temp[cur.specName]=e2;
                                temp.sku_index +=i;
                                result.push(temp);
                            }
                        })
                        return result;
                    },[{}]);
                    //添加价格
                    res.forEach(e1=>{
                        e1.price = 0;
                        e1.availableStock = 0;
                        e1.sku_index = e1.sku_index.substring(1);
                    })
                    this.skus = res;
                },
                deep:true
            }
        }
    }

</script>

<style scoped>

</style>