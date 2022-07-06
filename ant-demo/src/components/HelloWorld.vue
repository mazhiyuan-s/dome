<template>
  <a-button class="editable-add-btn" style="margin-bottom: 8px" @click="handleAdd('add')">Add</a-button>
  <a-table bordered :data-source="dataSource" :columns="columns">
    <template #bodyCell="{ column, text, record }">
      <template v-if="column.dataIndex === 'name'">
        <div class="editable-cell">
          <div v-if="editableData[record.key]" class="editable-cell-input-wrapper">
            <a-input v-model:value="editableData[record.key].name" @pressEnter="save(record.key)" />
            <check-outlined class="editable-cell-icon-check" @click="save(record.key)" />
          </div>
          <div v-else class="editable-cell-text-wrapper">
            {{ text || ' ' }}
            <edit-outlined class="editable-cell-icon" @click="edit(record.key)" />
          </div>
        </div>
      </template>
      <template v-else-if="column.dataIndex === 'operation'">
        <a-popconfirm
          v-if="dataSource.length"
          title="Sure to delete?"
          @confirm="onDelete(record.key)"
        >
          <a>Delete</a>
        </a-popconfirm>
        <a-button  style="margin-bottom: 8px" @click="handleEdit(record)" type="link">编辑</a-button>
      </template>
    </template>
  </a-table>
  <!-- 添加框 -->
  <div>
    <a-modal
      v-model:visible="visible"
      title="添加数据"
      width="100%"
      wrap-class-name="full-modal"
      @ok="handleOk"
    >
      <a-form
        :model="formState"
        name="basic"
        :label-col="{ span: 4 }"
        :wrapper-col="{ span: 16 }"
        autocomplete="off"
        @finish="onFinish"
        @finishFailed="onFinishFailed"
      >
        <a-form-item
          label="name"
          name="name"
          :rules="[{ required: true, message: 'Please input your name!' }]"
        >
          <a-input v-model:value="formState.name" />
        </a-form-item>

        <a-form-item
          label="type"
          name="type"
        >
          <a-select
            v-model:value="formState.type"
            placeholder="Please select"
            :options="options"
          ></a-select>
        </a-form-item>
        <a-form-item
          label="required"
          name="required"
        >
          <a-radio-group v-model:value="formState.required" name="radioGroup">
            <a-radio :value="'是'">是</a-radio>
            <a-radio :value="'否'">否</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item
          v-if="formState.type == '单行文本'"
          label="option"
          name="option"
        >
          <a-input v-model:value="formState.option" />
        </a-form-item>

        <a-form-item
          v-if="formState.type == '日期'"
          label="option"
          name="option"
        > 
          <a-form-item
            name="dataType"
          >
            <a-radio-group v-model:value="formState.dataType" name="radioGroup" @change="dataTypeChenge">
              <a-radio :value="'年'">年</a-radio>
              <a-radio :value="'年-月'">年-月</a-radio>
              <a-radio :value="'年-月-日'">年-月-日</a-radio>
              <a-radio :value="'年-月-日 时：分：秒'">年-月-日 时：分：秒</a-radio>
            </a-radio-group>
            <!-- <a-date-picker v-model:value="formState.option" :picker="formState.dataType? '年' : 'year' ? '年-月' : 'month' ? '年-月-日' : 'Select Time' " /> -->
            <a-date-picker v-if="formState.dataType == '年'" :picker="'year' "  @change="onChange" @ok="onOk"/>
            <a-date-picker v-if="formState.dataType == '年-月'" :picker="'month' "  @change="onChange" @ok="onOk"/>
            <a-date-picker v-if="formState.dataType == '年-月-日'" @change="onChange" @ok="onOk"/>
            <a-date-picker v-if="formState.dataType == '年-月-日 时：分：秒'" show-time :placeholder="'Select Time' "  @change="onChange" @ok="onOk"/>
          </a-form-item>

        </a-form-item>

        <a-form-item
          v-if="formState.type == '下拉'"
          label="option"
          name="option"
        > 
          <a-select
            mode="multiple"
            style="width: 100%"
            placeholder="Please select"
            :options="[...Array(25)].map((_, i) => ({ value: '选项' + (i + 1) }))"
            @change="handleChange"
          ></a-select>
        </a-form-item>

        <a-form-item :wrapper-col="{ offset: 8, span: 16 }">
          <a-button v-if="buttonType == 'add'" type="primary" html-type="submit">提交</a-button>
          <a-button v-if="buttonType == 'edit'" type="primary" html-type="submit">提价</a-button>
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>
<script>
import { computed, defineComponent, reactive, ref } from 'vue';
import { CheckOutlined, EditOutlined } from '@ant-design/icons-vue';
import { cloneDeep } from 'lodash-es';

export default defineComponent({
  components: {
    CheckOutlined,
    EditOutlined,
  },

  setup() {
    const buttonType = ref('add')
    const keyTable = ref('0')
    const options = reactive([{
      value: '单行文本',
    },{
      value: '日期',
    },{
      value: '下拉',
    }])
    const formState = reactive({
      name: 'Edward King 0',
      type: '日期',
      required: '否',
      option: null,
      dataType: ''
    });
    const visible = ref(false);
    const columns = [{
      title: '字段名称',
      dataIndex: 'name',
      width: '30%',
    }, {
      title: '字段类型',
      dataIndex: 'type',
    }, {
      title: '是否必填',
      dataIndex: 'required',
    }, {
      title: '字段选项',
      dataIndex: 'option',
    }, {
      title: 'operation',
      dataIndex: 'operation',
    }];
    const dataSource = ref([{
      key: '0',
      name: 'Edward King 0',
      type: '单行文本',
      required: '是',
      option: '选项1,选项2',
      dataType: '年'
    }]);
    const count = computed(() => dataSource.value.length + 1);
    const editableData = reactive({});

    const edit = key => {
      editableData[key] = cloneDeep(dataSource.value.filter(item => key === item.key)[0]);
    };

    const save = key => {
      Object.assign(dataSource.value.filter(item => key === item.key)[0], editableData[key]);
      delete editableData[key];
    };

    const onDelete = key => {
      dataSource.value = dataSource.value.filter(item => item.key !== key);
    };
    const dataTypeChenge = values =>{
      dataSource.value.dataType = values.target.value
    }

    const handleChange = (value) => {
      formState.option = String(value)
    };
    const onChange = (value, dateString) => {
      formState.option = dateString
    };

    const onOk = (value) => {
      console.log('onOk: ', value);
    };
    const handleAdd = () => {
      visible.value = true;
      buttonType.value = "add"

    };
    const handleEdit = key  => {
      visible.value = true;
      console.log(key)
      keyTable.value=key
      formState.name = key.name
      formState.type = key.type
      formState.required = key.required
      formState.option = key.option
      buttonType.value = "edit"
    };
    // const editSub = () =>{
    //   let key = keyTable.value.key
    //   console.log(key)
    //   visible.value = false;
    // }
    const onFinish = values => {
      console.log('Success:', values);
      if(buttonType.value == 'add'){
        const newData = {
          key: `${count.value}`,
          name: values.name,
          type: values.type,
          required: values.required,
          option: values.option,
          dataType: values.dataType,
        };
        dataSource.value.push(newData);
      }else if(buttonType.value == 'edit'){
        console.log("aaa")
        let key = keyTable.value.key
        dataSource.value.map( (item , index)=>{
          if(item.key == key){
            console.log(item,index)
            dataSource.value.splice(index,1,values)
            console.log("aaa====",dataSource.value)
          }
        })
      }
      visible.value = false;
      // formState.name = ''
      // formState.type = ''
      // formState.required = ''
      formState.option = ''
    };
    
    return {
      keyTable,
      buttonType,
      handleChange,
      onOk,
      onChange,
      dataTypeChenge,
      options,
      handleEdit,
      onFinish,
      formState,
      visible,
      columns,
      onDelete,
      handleAdd,
      dataSource,
      editableData,
      count,
      edit,
      save,
    };
  },

});
</script>
<style lang="less">
.editable-cell {
  position: relative;
  .editable-cell-input-wrapper,
  .editable-cell-text-wrapper {
    padding-right: 24px;
  }

  .editable-cell-text-wrapper {
    padding: 5px 24px 5px 5px;
  }

  .editable-cell-icon,
  .editable-cell-icon-check {
    position: absolute;
    right: 0;
    width: 20px;
    cursor: pointer;
  }

  .editable-cell-icon {
    margin-top: 4px;
    display: none;
  }

  .editable-cell-icon-check {
    line-height: 28px;
  }

  .editable-cell-icon:hover,
  .editable-cell-icon-check:hover {
    color: #108ee9;
  }

  .editable-add-btn {
    margin-bottom: 8px;
  }
}
.editable-cell:hover .editable-cell-icon {
  display: inline-block;
}
// 添加弹框css
.full-modal {
  .ant-modal {
    max-width: 100%;
    top: 0;
    padding-bottom: 0;
    margin: 0;
  }
  .ant-modal-content {
    display: flex;
    flex-direction: column;
    height: calc(100vh);
  }
  .ant-modal-body {
    flex: 1;
  }
}
</style>