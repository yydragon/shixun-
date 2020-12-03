<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i> 基础表格
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button
                    type="primary"
                    icon="el-icon-delete"
                    class="handle-del mr10"
                    @click="delAllSelection"
                >批量删除</el-button>
                 <el-button type="primary" @click="addOrder()">添加订单</el-button>
                <el-select v-model="query.orderStatus" placeholder="状态" @change="selectChange" class="handle-select mr10">
                    <el-option key="1" label="未接单" value=10></el-option>
                    <el-option key="2" label="已接单" value=20></el-option>
                    <el-option key="3" label="已拒绝" value=30></el-option>
                    <el-option key="4" label="生产中" value=40></el-option>
                    <el-option key="5" label="订单完成" value=50></el-option>
                </el-select>
                <!-- <el-input v-model="query.productName" placeholder="产品名称" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button> -->
                <el-button type="primary" icon="el-icon-search" @click="restart">重置</el-button>

            </div>
            <el-table
                :data="backData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
                <el-table-column align="center" prop="orderSeq" label="订单编号"></el-table-column>
                <el-table-column align="center" prop="orderSource" label="订单来源"></el-table-column>
                
                <el-table-column prop="productId" label="产品ID"  align="center"></el-table-column>
                <el-table-column prop="productCount" label="产品数量" align="center"></el-table-column>
                <el-table-column prop="endDate" align="center" label="订单截止时间" :formatter="dateFormat"></el-table-column>
                
                <el-table-column prop="orderStatus" label="订单状态" align="center">
                    <template slot-scope="scope">
                        <el-tag
                            v-if="scope.row.orderStatus==10"
                        >{{"未接单"}}</el-tag>
                        <el-tag
                            v-if="scope.row.orderStatus==20"
                        >{{"已接单"}}</el-tag>
                        <el-tag
                            v-if="scope.row.orderStatus==30"
                        >{{"已拒绝"}}</el-tag>
                        <el-tag
                            v-if="scope.row.orderStatus==40"
                        >{{"生产中"}}</el-tag>
                        <el-tag
                            v-if="scope.row.orderStatus==50"
                        >{{"订单完成"}}</el-tag>
                    </template>
                    
                </el-table-column>
                <el-table-column label="开关" align="center">
                    <template slot-scope="scope">
                    <el-switch
                        v-model="scope.row.orderStatus"
                        :active-value="20"
                        :inactive-value="10"
                        active-color="#02538C"
                        inactive-color="#B9B9B9"
                        @change="changeSwitch(scope.row)"/>
                    
                    </template>
                    
                </el-table-column>

                <el-table-column prop="factoryId" align="center" label="工厂ID"></el-table-column>
                <el-table-column prop="createTime" align="center" label="创建时间" :formatter="dateFormat"></el-table-column>

                <el-table-column prop="updateTime" align="center" label="修改时间" :formatter="dateFormat"></el-table-column>

                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button
                            type="text"
                            icon="el-icon-edit"
                            @click="handleEdit(scope.$index, scope.row)"
                            v-show="scope.row.orderStatus==10 || scope.row.orderStatus==20 ||scope.row.orderStatus==30"
                        >编辑</el-button>
                        <el-button
                            type="text"
                            icon="el-icon-delete"
                            class="red"
                            @click="handleDelete(scope.$index, scope.row)"
                        >删除</el-button>
                        <el-button size="mini" type="success" v-show="scope.row.orderStatus==40" icon="el-icon-check" circle @click="changeOrderStatusSuccess(scope.$index, scope.row)"></el-button>
                        <el-button size="mini" type="warning" v-show="scope.row.orderStatus==50" icon="el-icon-close" circle @click="changeOrderStatusQuitSuccess(scope.$index, scope.row)"></el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination
                    background
                    layout="total, prev, pager, next"
                    :current-page="pagequery.pageIndex"
                    :page-size="pagequery.pageSize"
                    :total="pageTotal"
                    @current-change="handlePageChange"
                ></el-pagination>
            </div>
        </div>

        <!-- 添加弹出框 -->
        <el-dialog title="添加订单" :visible.sync="addVisible" width="30%">
            <el-form ref="form" :model="form" label-width="100px">
                <el-form-item label="订单来源">
                     <el-select v-model="form.orderSource" placeholder="请选择状态">
                    <el-option label="线下" value="线下"></el-option>
                    </el-select>
                </el-form-item>
                
                <el-form-item label="产品名称">
                    <el-select v-model="form.productId" placeholder="请选择" @visible-change="selectProduct($event)" @change="AfterSelect" >
                        <el-option
                        v-for="(item,index) in this.selectData"
                        :key="index"
                        :label="item.productName"
                        :value="item.productNum"
                        >
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="产品数量">
                    <el-input v-model="form.productCount"></el-input>
                </el-form-item>
                <el-form-item label="工厂ID">
                    <el-input v-model="form.factoryId"></el-input>
                </el-form-item>
                
                <el-form-item label="订单状态">
                    <el-select v-model="form.orderStatus">
                    <el-option label="未接单" value=10></el-option>              
                    </el-select>
                </el-form-item>
                <el-form-item label="订单截止日期" >
                    <el-col :span="11">
                    <el-date-picker type="date" placeholder="选择日期" v-model="form.endDate"  style="width: 160%;"></el-date-picker>
                    </el-col>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveAdd">确 定</el-button>
            </span>
        </el-dialog>

<!-- 编辑弹出框 -->
        <el-dialog title="修改订单" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="editForm" label-width="100px">
                <el-form-item label="订单来源">
                    <el-input v-model="editForm.orderSource"></el-input>
                </el-form-item>
                
                <el-form-item label="产品名称">
                    <el-select v-model="editForm.productId" disabled   @visible-change="selectProduct($event)" @change="AfterSelect" >
                        <el-option
                        v-for="(item) in this.selectData"
                        :key="item.productNum"
                        :label="item.productName"
                        :value="item.productNum"
                        >
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="产品数量">
                    <el-input v-model="editForm.productCount"></el-input>
                </el-form-item>
                <el-form-item label="工厂ID">
                    <el-input v-model="editForm.factoryId"></el-input>
                </el-form-item>

                <el-form-item label="订单截止日期" >
                    <el-col :span="11">
                    <el-date-picker type="date" placeholder="选择日期" v-model="form.endDate"  style="width: 160%;"></el-date-picker>
                    </el-col>
                </el-form-item>
                
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
        
    </div>
</template>

<script>
import { fetchData } from '../../../api/index';
import moment from 'moment'
export default {
    name: 'basetable',
    data() {
        return {
            query: {
                productName: '',
                orderStatus:'',
                pageIndex: 1,
                pageSize: 10
            },
            pagequery:{
                pageIndex: 1,
                pageSize: 10
            },
            tableData: [],
            value:"",
            multipleSelection: [],
            delList: [],
            addVisible: false,
            editVisible: false,
            pageTotal: 0,

            selectData:[],
            form: {
                orderSource:0,
                productId:"",
                productCount:"",
                factoryId:"",
                endDate:"",
                orderStatus:0
            },
            editForm:{
                orderSource:"",
                productId:"",
                productCount:"",
                factoryId:"",
                endDate:"",
                productName:"",
                orderStatus:0
            },
            idx: -1,
            id: -1,
            backData:[]
        };
    },
    created() {
        this.getData();
        this.getData();
    },
    watch: {  //监听
      $route(to, from) { //路由变化方式，路由发生变化，方法就会执行
        this.getData();
        
      }
    },
    mounted(){
        this.getData();
    },
    methods: {  
        changeOrderStatusQuitSuccess(index, row){
            row.orderStatus = 40
            this.changeSwitch(row) 
        },
        changeOrderStatusSuccess(index, row){
            row.orderStatus = 50
           this.changeSwitch(row) 
            
        },
        dateFormat:function(row,column){
            var date = row[column.property];
        if(date == undefined){
            return ''
        } ;
        return moment(date).format("YYYY-MM-DD HH:mm:ss")
        },
        changeSwitch(row){
            console.log(row)
             this.$axios.post("/api/order/switch",row)
             .then(response=>{
                 this.getData();
             })
            
        },
        AfterSelect(val){
            console.log("name"+val)
            this.form.productId = val
            //this.editForm.productId = val
            
            // this.$axios.get("/api/order/getProductName",{
            //     params:{
            //         val:val
            //     }
            // })
            // .then(response=>{
            //     console.log(response.data.data.name)
            //     this.editForm.productName = response.data.data.name
            // })
        },
        selectProduct(callback){
            console.log(callback)
            if(callback){
            this.$axios.get("/api/equipment-product/getProduct")
            .then(response=>{
                this.selectData = response.data.data.data.records
                console.log(this.selectData)
            })
            }
            
        },
        restart(){
            this.query.orderStatus = ""
            this.query.name = ""
            this.getData()
        },
        saveAdd(){
            this.$axios.post("/api/order/add",this.form)
            .then(response=>{
                this.addVisible = false
                this.getData()
            })
        },
        addOrder(){
            this.form={}
            this.addVisible = true;

        },
        // 获取 easy-mock 的模拟数据
        getData() {
            this.$axios.get("/api/order/list",{
                params:{
                    val:this.pagequery.pageIndex
                }
            })
            .then(response=>{
                console.log(response.data.data.data.records)
                this.backData = response.data.data.data.records
                this.pageTotal = response.data.data.data.total
            })
        },
        selectChange(val){
            console.log(val)
            this.$axios.get("/api/order/selectQuery",{
                params:{
                    val:val
                }
            }).then(response=>{
                this.backData = response.data.data.data.records
                this.pageTotal = response.data.data.data.total
            })
        },
        // 触发搜索按钮
        handleSearch() {
            this.$axios.post("/api/order/query",this.query)
            .then(response=>{
                this.backData = response.data.data.data.records
                this.pageTotal = response.data.data.data.total
                
            })
            
        },
        // 删除操作
        handleDelete(index, row) {
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            }).then(() => {
                    this.$axios.get("/api/order/delete",{
                        params:{
                            orderSeq : row.orderSeq
                        }
                    })
                    
                    this.$message.success('删除成功');
                    this.tableData.splice(index, 1);
                    this.getData()
                    this.getData()
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });          
                    });
                    },
        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
            
        },
        delAllSelection() {
            const length = this.multipleSelection.length;
            let str = '';
            //this.delList = this.delList.concat(this.multipleSelection);
            this.delList = this.multipleSelection.map(item => item.equipmentSeq)
            this.$axios.post("/api/order/deleteBatch",this.delList)
            .then(response=>{
                this.getData()
            })
            console.log("p删除"+this.delList)
            console.log("m删除"+this.multipleSelection)
            // for (let i = 0; i < length; i++) {
            //     str += this.multipleSelection[i].equipmentName + ' ';
            // }
            this.$message.success(`删除成功`);
            this.multipleSelection = [];
            
        },
        // 编辑操作
        handleEdit(index, row) {
            this.$axios.get("/api/product/list")
            .then(response=>{
                this.selectData = response.data.data.data.records
                console.log(this.selectData)
            })
            this.idx = index;
            this.form = row;
            this.editVisible = true;
            this.editForm = this.form
            console.log(this.editForm)
        },
        // 保存编辑
        saveEdit() {
            this.$axios.post("/api/order/edit",this.editForm)
            .then(()=>{
                this.editVisible = false;
                this.$message.success(`修改第 ${this.idx + 1} 行成功`);
                this.$set(this.tableData, this.idx, this.editForm);
            })
            
        },
        // 分页导航
        handlePageChange(val) {
            this.pagequery.pageIndex = val
             this.getData()
        }
    }
};
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}
.handle-select {
    width: 120px;
}
.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>

