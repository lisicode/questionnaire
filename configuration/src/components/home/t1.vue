<template>
  <div class="t1">
    <el-button type="primary" plain @click="dialogFormVisible = true">新增题目</el-button>
    <el-button type="primary" :disabled="examination.length === 0" @click="dialogVisible = true">生成试卷</el-button>
    <el-table stripe :data="examination">
      <el-table-column type="expand">
        <template slot-scope="props">
          <p v-for="o in props.row.list">
            <el-tag type="danger">备选：</el-tag>
            <el-tag>{{ o.option }}</el-tag>
          </p>
        </template>
      </el-table-column>
      <el-table-column
          label="类型"
          prop="type">
        <template slot-scope="scope">
          <el-tag :type="scope.row.type === '01' ? 'primary' : 'success'">{{ scope.row.type==='01' ? '单选题' : '多选题' }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column
          label="题目"
          prop="topic">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" type="danger" @click="removeExamination(scope.$index)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="新增题目" :visible.sync="dialogFormVisible" width="50%">
      <el-form :model="question" status-icon :rules="rules" ref="question">
        <el-form-item prop="type">
          <el-select v-model="question.type" placeholder="题目类型">
            <el-option label="单项选择" value="01"></el-option>
            <el-option label="多项选择" value="02"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item prop="topic">
          <el-input v-model="question.topic" placeholder="请输入题目"></el-input>
        </el-form-item>
        <el-form-item v-for="(i, index) in question.list" :prop="'list.' + index + '.option'" :rules="{required: true, message: '请输入题目', trigger: 'blur'}">
          <el-input placeholder="请输入备选项" v-model="i.option">
            <el-button slot="append" :disabled="index < 2" @click="removeQuestion(index)">删除</el-button>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="add" plain>增加备选项</el-button>
          <el-button type="primary" @click="submitQuestion('question')">生成题目</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <el-dialog title="生成试卷" :visible.sync="dialogVisible" width="50%">
      <el-form :model="examinationData" :rules="rules" ref="examinationData">
        <el-form-item prop="title">
          <el-input v-model="examinationData.title" placeholder="请为试卷命名"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitExamination('examinationData')">提交</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {Request, ApiConfig, GetLocalStorage} from '@/assets/js/config'

export default {
  name: 't1',
  data() {
    return {
      question: {
        type: '',
        topic: '',
        values: [],
        list: [{option: ''}, {option: ''}],
      },
      rules: {
        type: [{required: true, message: '请选择题目类型', trigger: 'change'},],
        topic: [{required: true, message: '请输入题目', trigger: 'blur'},],
        title: [{required: true, message: '请为试卷命名', trigger: 'blur'},],
      },
      dialogVisible: false,
      dialogFormVisible: false,
      examination: [],
      examinationData: {
        title: ''
      },
    }
  },
  created() {
    console.log(GetLocalStorage('userData'))
  },
  methods: {
    add() {
      this.question.list.push({option: ''})
    },
    removeQuestion(e) {
      this.question.list.splice(e, 1)
    },
    submitQuestion(question) {
      this.$refs[question].validate((valid) => {
        if (valid) {
          let obj = JSON.parse(JSON.stringify(this.question));
          this.examination.push(obj);
          console.log(this.examination)
        } else {
          return false;
        }
      });
    },
    removeExamination(e) {
      this.examination.splice(e, 1)
    },
    submitExamination(examinationData) {
      this.$refs[examinationData].validate((valid) => {
        if (valid) {
          this.dialogVisible = false;
          Request({
            method: 'post',
            data: {
              api: ApiConfig.examinationPaper,
              account: GetLocalStorage('userData').account,
              department: GetLocalStorage('userData').department,
              title: this.examinationData.title,
              examination: this.examination
            }
          }).then(res => {
            console.log(res)
          })
        } else {
          return false;
        }
      });
    },
  }
}
</script>

<style lang="scss" scoped>

.el-form {
  .el-select {
    width: 100%;
  }
}
.el-table {
  margin-top: 20px;
  p {
    .el-tag {
      margin: 5px;
    }
  }
}

</style>
