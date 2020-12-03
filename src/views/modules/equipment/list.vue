<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i> 设备管理
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
                 <el-button type="primary" @click="addProduct()">添加设备</el-button>
                <el-select v-model="query.flag" placeholder="状态" @change="selectChange" class="handle-select mr10">
                    <el-option key="1" label="启用" value=10></el-option>
                    <el-option key="2" label="停用" value=20></el-option>
                    <el-option key="3" label="故障" value=30></el-option>
                </el-select>
                <el-input v-model="query.name" placeholder="设备名称" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
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
                <el-table-column prop="equipmentSeq" align="center" label="设备序号"></el-table-column>
                <el-table-column prop="equipmentName" align="center" label="设备名称"></el-table-column>
                <!-- <el-table-column label="账户余额">
                    <template slot-scope="scope">￥{{scope.row.money}}</template>
                </el-table-column> -->
                <el-table-column label="设备图片" prop="equipmentImgUrl" align="center" width="120px">
                    <template slot-scope="scope">
                        <img :src="scope.row.equipmentImgUrl" v-image-preview width="100px" height="100px">
                    </template>
                </el-table-column>
                <el-table-column prop="equipmentStatus" align="center" label="设备状态">
                    <template slot-scope="scope">
                        <el-tag
                            v-if="scope.row.equipmentStatus==10"
                        >{{"已启用"}}</el-tag>
                        <el-tag
                            v-if="scope.row.equipmentStatus==20"
                        >{{"未启用"}}</el-tag>
                        <el-tag
                            v-if="scope.row.equipmentStatus==30"
                        >{{"故障"}}</el-tag>
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
                        >编辑</el-button>
                        <el-button
                            type="text"
                            icon="el-icon-delete"
                            class="red"
                            @click="handleDelete(scope.$index, scope.row)"
                        >删除</el-button>
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

        <!-- 编辑弹出框 -->
        <el-dialog title="添加设备" :visible.sync="addVisible" width="35%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="设备名称">
                    <el-input v-model="form.equipmentName" style="width: 360px"></el-input>
                </el-form-item>
                
                <el-form-item label="设备图片">
                    <el-upload 
                        class="upload-demo"
                        drag
                        action="/api/product/upload"
                        :on-success="afterSuccess"
                        multiple>
                        <i class="el-icon-upload"></i>
                        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                        <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form-item>
                <el-form-item label="设备状态">
                    <el-select v-model="form.equipmentStatus" placeholder="请选择状态">
                    <el-option label="启用" :value="10"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="工厂ID">
                    <el-input v-model="form.factoryId"  style="width: 360px"></el-input>
                </el-form-item>

                
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveAdd">确 定</el-button>
            </span>
        </el-dialog>

<!-- 编辑弹出框 -->
        <el-dialog title="修改设备" :visible.sync="editVisible" width="35%">
            <el-form ref="form" :model="editForm" label-width="70px">
                <el-form-item label="设备名称">
                    <el-input v-model="editForm.equipmentName"  style="width: 360px"></el-input>
                </el-form-item>
                
                <el-form-item label="设备图片">
                    <img :src="editForm.equipmentImgUrl" v-image-preview width="100px" height="100px">
                        <el-upload 
                        class="upload-demo"
                        drag
                        action="/api/product/upload"
                        :on-success="afterSuccess2"
                        multiple>
                        <i class="el-icon-upload"></i>
                        <div class="el-upload__text">修改图片，将文件拖到此处，或<em>点击上传</em></div>
                        <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form-item>
                <el-form-item label="设备状态">
                    <el-select v-model="form.equipmentStatus" placeholder="请选择状态">
                    <el-option label="已启用" :value="10"></el-option>
                    <el-option label="未启用" :value="20"></el-option>
                    <el-option label="故障" :value="30"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="工厂ID">
                    <el-input v-model="editForm.factoryId"  style="width: 360px"></el-input>
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
                name: '',
                pageIndex: 1,
                pageSize: 10
            },
            pagequery:{
                pageIndex: 1,
                pageSize: 10
            },
            tableData: [],
            multipleSelection: [],
            delList: [],
            addVisible: false,
            editVisible: false,
            pageTotal: 0,
            form: {
                equipmentName:"",
                equipmentImgUrl:"",
                factoryId:"",
                equipmentStatus:"",
            },
            editForm:{
                equipmentName:"",
                equipmentImgUrl:"",
                factoryId:"",
                equipmentStatus:"",
            },
            idx: -1,
            id: -1,
            backData:[]
        };
    },
    created() {
        
        this.getData();
    },
    methods: {
        afterSuccess2(response, file, fileList){
            this.editForm.equipmentImgUrl = response.data.url.url
        },
        afterSuccess(response, file, fileList){
            this.form.equipmentImgUrl = response.data.url.url
        },
        restart(){
            this.query.flag = ""
            this.query.name = ""
            this.getData()
        },
        saveAdd(){
            this.$axios.post("/api/equipment/add",this.form)
            .then(response=>{
                this.addVisible = false
                this.getData()
            })
        },
        addProduct(){
            this.form={}
            this.addVisible = true;
        },
        // 获取 easy-mock 的模拟数据
        getData() {
            this.$axios.get("/api/equipment/list",{
                params:{
                    val:this.pagequery.pageIndex
                }
            })
            .then(response=>{
                console.log(response.data.data.data)
                this.backData = response.data.data.data.records
                this.pageTotal = response.data.data.data.total
            })
        },
        selectChange(val){
            console.log(val)
            this.$axios.get("/api/equipment/selectQuery",{
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
            this.$axios.post("/api/equipment/query",this.query)
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
                    this.$axios.get("/api/equipment/delete",{
                        params:{
                            equipmentSeq : row.equipmentSeq
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
            this.$axios.post("/api/equipment/deleteBatch",this.delList)
            .then(response=>{
                this.getData()
            })
            console.log("p删除"+this.delList)
            console.log("m删除"+this.multipleSelection)
            for (let i = 0; i < length; i++) {
                str += this.multipleSelection[i].equipmentName + ' ';
            }
            this.$message.success(`删除了${str}`);
            this.multipleSelection = [];
            
        },
        dateFormat:function(row,column){
            var date = row[column.property];
        if(date == undefined){
            return ''
        } ;
        return moment(date).format("YYYY-MM-DD HH:mm:ss")
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.editVisible = true;
            this.editForm = this.form
        },
        // 保存编辑
        saveEdit() {
            this.$axios.post("/api/equipment/edit",this.editForm)
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

