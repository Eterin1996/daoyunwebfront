<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-tickets"></i> 字典详情页面
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button
                        type="primary"
                        icon="el-icon-delete"
                        class="handle-del mr10"
                        @click="handleAdd"
                >新增字典</el-button>
                <el-tooltip class="item" effect="dark" placement="top-end">
                    <div slot="content">
                        id：{{this.dicId}}
                        <br />
                        详情：{{this.dicDescription}}
                        <br />
                        Code：{{this.code}}
                    </div>
                    <el-button>名称：{{this.dicName}}</el-button>
                </el-tooltip>
            </div>
            <el-table
                    :data="tableData"
                    border
                    class="table"
                    ref="multipleTable"
                    header-cell-class-name="table-header"
                    @selection-change="handleSelectionChange"
            >
                <el-table-column prop="id" label="内容编号" width="55" align="center"></el-table-column>
                <el-table-column prop="itemKey" label="ItemKey"></el-table-column>
                <el-table-column prop="itemValue" label="ItemValue"></el-table-column>
                <el-table-column prop="isDefault" label="是否默认"></el-table-column>
                <el-table-column prop="code" label="Code"></el-table-column>
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
                        :current-page="query.page"
                        :page-size="query.pageSize"
                        :total="selectTotal"
                        @current-change="handlePageChange"
                ></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="100px">
                <el-form-item label="Id">
                    <el-input v-model.number="form.id" disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="内容编号">
                    <el-input v-model.number="form.dicId" disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="ItemKey">
                    <el-input v-model.number.number="form.itemKey"></el-input>
                </el-form-item>
                <el-form-item label="ItemValue">
                    <el-input v-model="form.itemValue"></el-input>
                </el-form-item>
                <el-form-item label="是否默认(之后要改成selector)">
                    <el-input v-model.number="form.isDefault"></el-input>
                </el-form-item>
                <el-form-item label="Code">
                    <el-input v-model="form.code"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveEdit">确 定</el-button>
      </span>
        </el-dialog>

        <!-- 新增弹出框 -->
        <el-dialog title="新增" :visible.sync="addVisible" width="30%">
            <el-form ref="addform" :model="addform" label-width="100px">
                <el-form-item label="ItemKey">
                    <el-input v-model.number="addform.itemKey"></el-input>
                </el-form-item>
                <el-form-item label="ItemValue">
                    <el-input v-model="addform.itemValue"></el-input>
                </el-form-item>
                <el-form-item label="是否默认(之后要改成selector)">
                    <el-input v-model.number="addform.isDefault"></el-input>
                </el-form-item>
                <el-form-item label="Code">
                    <el-input v-model="addform.code"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
        <el-button @click="addVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveAdd">确 定</el-button>
      </span>
        </el-dialog>
    </div>
</template>

<script>
    import { fetchData } from "../../api/index";
    import axios from "axios";
    export default {
        name: "basetable",
        data() {
            return {
                query: {
                    page: 1,
                    pageSize: 10
                },
                tableData: [],
                selectTotal: 0,
                multipleSelection: [],
                delList: [],
                editVisible: false,
                addVisible: false,
                form: {
                    id: 0,
                    dicId: 0,
                    itemKey: 0,
                    itemValue: "",
                    isDefault: 0,
                    code: ""
                },
                addform: {
                    itemKey: 0,
                    itemValue: "",
                    isDefault: 0,
                    code: ""
                },
                dicId: 0,
                dicName: "",
                code: "",
                dicDescription: "",

                idx: -1,
                id: -1
            };
        },
        created() {},
        mounted() {},
        beforeRouteEnter(to, from, next) {
            next(vm => {
                console.log(vm);
                // 每次进入路由执行
                vm.initRouter();
                vm.getData();
                vm.getDataCount();
            });
        },
        methods: {
            // 获取 easy-mock 的模拟数据
            getData() {
                axios
                    .post(
                        "http://localhost:8080/daoyunWeb_war_exploded/DictionaryDetail/getDicdByPage",
                        {
                            page: this.query.page,
                            pageSize: this.query.pageSize,
                            dicId: this.dicId
                        },
                        { headers: { "Content-Type": "application/json" } }
                    )
                    .then(
                        res => {
                            console.log(res);
                            if (res.status == 200) {
                                this.tableData = res.data.data;
                            }
                        },
                        error => {
                            console.log(error);
                        }
                    );
            },
            initRouter() {
                // this.paperId=this.$route.params.paperId;
                // this.paperName=this.$route.params.paperName;
                // this.paperNum=this.$route.params.paperNum;
                // this.paperDetail=this.$route.params.paperDetail;

                this.dicId = parseInt(localStorage.getItem("dicId"));
                this.dicName = localStorage.getItem("dicName");
                this.code = localStorage.getItem("code");
                this.dicDescription = localStorage.getItem("dicDescription");
            },
            getDataCount() {
                //TODO 待加入搜索限定参数
                axios
                    .post(
                        "http://localhost:8080/daoyunWeb_war_exploded/DictionaryDetail/getDicdCount/" +
                        this.dicId
                    )
                    .then(
                        res => {
                            console.log(res);
                            if (res.status == 200) {
                                this.selectTotal = res.data.data;
                            }
                        },
                        error => {
                            console.log(error);
                        }
                    );
            },
            updateDicd() {
                axios
                    .post(
                        "http://localhost:8080/daoyunWeb_war_exploded/DictionaryDetail/updateDicdJson",
                        {
                            id: this.form.id,
                            dicId: this.form.dicId,
                            itemKey: this.form.itemKey,
                            itemValue: this.form.itemValue,
                            isDefault: this.form.isDefault,
                            code: this.form.code
                        },
                        { headers: { "Content-Type": "application/json" } }
                    )
                    .then(
                        res => {
                            console.log(res);
                            if (res.status == 200) {
                                this.getData();
                                this.getDataCount();
                            }
                        },
                        error => {
                            console.log(error);
                        }
                    );
            },
            addDicd() {
                axios
                    .post(
                        "http://localhost:8080/daoyunWeb_war_exploded/DictionaryDetail/addDicdJson",
                        {
                            dicId: this.dicId,
                            itemKey: this.addform.itemKey,
                            itemValue: this.addform.itemValue,
                            isDefault: this.addform.isDefault,
                            code: this.addform.code
                        },
                        { headers: { "Content-Type": "application/json" } }
                    )
                    .then(
                        res => {
                            console.log(res);
                            if (res.status == 200) {
                                this.getData();
                                this.getDataCount();
                            }
                        },
                        error => {
                            console.log(error);
                        }
                    );
            },
            deleteDicd() {
                axios
                    .post(
                        "http://localhost:8080/daoyunWeb_war_exploded/DictionaryDetail/deleteDicdJson/" +
                        this.form.id
                    )
                    .then(
                        res => {
                            console.log(res);
                            if (res.status == 200) {
                                this.getData();
                                this.getDataCount();
                            }
                        },
                        error => {
                            console.log(error);
                        }
                    );
            },
            // 触发搜索按钮
            handleSearch() {
                this.$set(this.query, "pageIndex", 1);
                this.getData();
            },
            // 删除操作
            handleDelete(index, row) {
                // 二次确认删除
                this.idx = index;
                this.form = row;
                this.$confirm("确定要删除吗？", "提示", {
                    type: "warning"
                })
                    .then(() => {
                        this.deleteDicd();
                        this.$message.success("删除成功");
                        //this.tableData.splice(index, 1);
                    })
                    .catch(() => {});
            },
            // 多选操作
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            delAllSelection() {
                const length = this.multipleSelection.length;
                let str = "";
                this.delList = this.delList.concat(this.multipleSelection);
                for (let i = 0; i < length; i++) {
                    str += this.multipleSelection[i].name + " ";
                }
                this.$message.error(`删除了${str}`);
                this.multipleSelection = [];
            },
            // 编辑操作
            handleEdit(index, row) {
                this.idx = index;
                this.form = row;
                this.editVisible = true;
            },
            // 保存编辑
            saveEdit() {
                this.editVisible = false;
                this.$message.success(`修改第 ${this.idx + 1} 行成功`);
                this.updateDicd();
                //this.$set(this.tableData, this.idx, this.form);
            },
            // 新增操作
            handleAdd() {
                this.addVisible = true;
            },
            // 保存新增
            saveAdd() {
                this.addDicd();
                this.addVisible = false;
            },
            // 分页导航
            handlePageChange(val) {
                this.$set(this.query, "page", val);
                this.getData();
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
