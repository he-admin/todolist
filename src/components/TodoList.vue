<template>
  <div class="todo-list">
    <header>
      todolist
    </header>

    <div class="form" v-if="showForm">
      <a-form :form="form" @submit="handleSubmit">
        <a-form-item label="名字" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input
            v-decorator="['name', { rules: [{ required: true, message: '请输入名字！' }] }]"
          />
        </a-form-item>
        <a-form-item label="年龄" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input
            v-decorator="['age', { rules: [{ required: true, message: '请输入年龄！' }] }]"
          />
        </a-form-item>
        <a-form-item label="地址" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input
            v-decorator="['address' +
             '', { rules: [{ required: true, message: '请输入地址！' }] }]"
          />
        </a-form-item>

        <a-form-item :wrapper-col="{ span: 12, offset: 5 }">
          <a-button @click="hidden">取消</a-button>
          <a-button type="primary" html-type="submit" style="margin-left:10px">
            添加
          </a-button>
        </a-form-item>
      </a-form>
    </div>


    <a-button class="editable-add-btn" style="margin-bottom: 10px" @click="show">添加</a-button>
    <a-table :columns="columns" :dataSource="data" bordered>
      <template
        v-for="col in ['name', 'age', 'address']"
        :slot="col"
        slot-scope="text, record, index"
      >
        <div :key="col">
          <a-input
            v-if="record.editable"
            style="margin: -5px 0"
            :value="text"
            @change="e => handleChange(e.target.value, record.key, col)"
          />
          <template v-else
          >{{text}}
          </template
          >
        </div>
      </template>
      <template slot="operation" slot-scope="text, record, index">
        <div class="editable-row-operations">
        <span v-if="record.editable">
          <a @click="() => save(record.key)">保存</a>

            <a @click="cancel(record.key)">取消</a>

        </span>
          <span v-else>
          <a @click="() => edit(record.key)">编辑</a>

          <a href="javascript:;" @click="onDelete(record.key)">删除</a>
        </span>
        </div>
      </template>
    </a-table>
  </div>

</template>
<script>
  const columns = [
    {
      title: '名字',
      dataIndex: 'name',
      width: '25%',
      scopedSlots: {customRender: 'name'},
    },
    {
      title: '年龄',
      dataIndex: 'age',
      width: '15%',
      scopedSlots: {customRender: 'age'},
    },
    {
      title: '地址',
      dataIndex: 'address',
      width: '40%',
      scopedSlots: {customRender: 'address'},
    },
    {
      title: '操作',
      dataIndex: 'operation',
      scopedSlots: {customRender: 'operation'},
    },
  ];

  export default {
    data() {
      // this.cacheData = this.data.map(item => ({...item}));
      return {
        data: JSON.parse(localStorage.getItem('item')),
        cacheData: [],
        columns,
        showForm: false,
        form: this.$form.createForm(this, {name: 'coordinated'}),
      };
    },
    created() {
      this.cacheData = this.data.map(item => ({...item}));
      if (this.data.length === 0) {
        this.data = [{key: 1, name: '示例', age: 18, address: '成都文理学院'}]
      }
    },
    methods: {
      show() {
        this.showForm = true;
      },
      hidden() {
        this.showForm = false;
      },
      handleSubmit(e) {
        e.preventDefault();
        this.form.validateFields((err, values) => {
          // 找到最后一个元素的key值，然后key值加一，赋给新的值。
          this.data.push({key: parseInt(this.data[this.data.length - 1].key) + 1, ...values});
          console.log(this.data)
          localStorage.setItem('item', JSON.stringify(this.data))
          if (!err) {
            console.log('Received values of form: ', values);
            this.showForm = false;
          }

        });
      },
      handleChange(value, key, column) {
        const newData = [...this.data];
        const target = newData.filter(item => key === item.key)[0];
        if (target) {
          target[column] = value;
          this.data = newData;
        }
      },
      onDelete(key) {
        const data = [...this.data];
        this.data = data.filter(item => item.key !== key);
        localStorage.setItem('item', JSON.stringify(this.data));
      },
      edit(key) {
        const newData = [...this.data];
        const target = newData.filter(item => key === item.key)[0];
        if (target) {
          target.editable = true;
          this.data = newData;
        }
      },
      save(key) {
        const newData = [...this.data];
        const target = newData.filter(item => key === item.key)[0];
        if (target) {
          delete target.editable;
          this.data = newData;
          this.cacheData = newData.map(item => ({...item}));
        }
        localStorage.setItem('item', JSON.stringify(this.data))
      },
      cancel(key) {
        const newData = [...this.data];
        const target = newData.filter(item => key === item.key)[0];
        if (target) {
          Object.assign(target, this.cacheData.filter(item => key === item.key)[0]);
          delete target.editable;
          this.data = newData;
        }
      },
    },
  };
</script>
<style scoped>
  header {
    font-size: 45px;
    color: #666666;
  }

  .todo-list {
    padding: 0 20px;
  }

  .editable-row-operations a {
    margin-right: 8px;
  }

  .form {
    position: relative;
  }

  .ant-form {
    position: absolute;
    left: 25%;
    top: 50px;
    z-index: 999;
    background-color: #F0F0F0;
    width: 50%;
  }
</style>
