<script lang="ts" setup>
import type {Ref, UnwrapRef} from 'vue';
import {computed, reactive, ref} from 'vue';
import {CheckOutlined, EditOutlined, PoweroffOutlined, CheckCircleFilled} from '@ant-design/icons-vue';
import {cloneDeep, toNumber} from 'lodash-es';
import { message } from 'ant-design-vue';

//模型运行日志
interface DataItem_result {
  key: string;
  result: string;
}

const columns_result = [
  {
    title: '模型运行日志',
    dataIndex: 'result',
  },
];

const dataSource_result: Ref<DataItem_result[]> = ref([]);
const count_result = computed(() => dataSource_result.value.length + 1);
const handleAdd_result = (something) => {
  const newData_result = {
    key: `${count_result.value}`,
    result: something,
  };
  dataSource_result.value.push(newData_result);
};

//抽屉设置
let model_state = ref("loading")
const open = ref(false);
//打开关闭时触发
const afterOpenChange = bool => {
  console.log('open', bool);
  if (bool) {} else {
    model_state.value = "loading"
    dataSource_result.value = []
  }
};
//打开抽屉
const showDrawer = () => {
  open.value = true;
};

//保存模型
let model_name = ref('MyModel')
let save_model = ref("F")

//使用已有模型设置
//模型列表
let model_list = ref([])
const value = ref(0);
const radioStyle = reactive({
  display: 'flex',
  height: '30px',
  lineHeight: '30px',
});

//设置列表
//预测数据列表
interface DataItem_next {
  key: string;
  x: number;
}

const columns_next = [
  {
    title: '需要预测值的自变量X',
    dataIndex: 'x',
    width: '30%',
  },
  {
    title: '操作',
    dataIndex: 'operation',
  },
];
const dataSource_next: Ref<DataItem_next[]> = ref([
  {
    key: '1',
    x: 1,
  },
  {
    key: '2',
    x: 2,
  },
]);
const count_next = computed(() => dataSource_next.value.length + 1);
const editableData_next: UnwrapRef<Record<string, DataItem_next>> = reactive({});

const edit_next = (key: string) => {
  editableData_next[key] = cloneDeep(dataSource_next.value.filter(item => key === item.key)[0]);
};
const save_next = (key: string) => {
  Object.assign(dataSource_next.value.filter(item => key === item.key)[0], editableData_next[key]);
  delete editableData_next[key];
};

const onDelete_next = (key: string) => {
  dataSource_next.value = dataSource_next.value.filter(item => item.key !== key);
};
const handleAdd_next = () => {
  const newData_next = {
    key: `${count_next.value}`,
    x: toNumber(`${count_next.value}`),
  };
  dataSource_next.value.push(newData_next);
};

//训练数据列表
interface DataItem {
  key: string;
  x: number;
  y: number;
}

const columns = [
  {
    title: '自变量X',
    dataIndex: 'x',
    width: '30%',
  },
  {
    title: '因变量Y',
    dataIndex: 'y',
    width: '30%',
  },
  {
    title: '操作',
    dataIndex: 'operation',
  },
];
const dataSource: Ref<DataItem[]> = ref([
  {
    key: '1',
    x: 1,
    y: 2,
  },
  {
    key: '2',
    x: 2,
    y: 3,
  },
]);

console.log(JSON.stringify(dataSource.value))

const count = computed(() => dataSource.value.length + 1);
const editableData: UnwrapRef<Record<string, DataItem>> = reactive({});

const edit = (key: string) => {
  editableData[key] = cloneDeep(dataSource.value.filter(item => key === item.key)[0]);
};
const save = (key: string) => {
  Object.assign(dataSource.value.filter(item => key === item.key)[0], editableData[key]);
  delete editableData[key];
};

const onDelete = (key: string) => {
  dataSource.value = dataSource.value.filter(item => item.key !== key);
};
const handleAdd = () => {
  const newData = {
    key: `${count.value}`,
    x: toNumber(`${count.value}`),
    y: toNumber(`${count.value + 1}`),
  };
  dataSource.value.push(newData);
};

//设置单选框
const mode = ref('predict');
const predict_mode = ref("LinearRegression")

//设置数字填充框
const learning_rate = ref(0.01)
const num_epochs = ref(100)
const degree = ref(2)

//高度自适应
function autoheight(id) {
  function adjustheight(id) {
    //获取卡片们的高度
    let predict = document.getElementById("predict").offsetHeight
    let call = document.getElementById("call").offsetHeight
    let save = document.getElementById("save");
    let save_height;
    //save没有消失
    if (save !== null) {
      save_height = save.offsetHeight;
      console.log(predict, call, save_height)
      // 获取指定div元素对象
      let divElement = document.getElementById(id);
      //document.documentElement.clientHeight
      divElement.style.height = (predict + call + save_height + 190).toString() + "px";
    } else {
      //save消失了
      let divElement = document.getElementById(id);
      //document.documentElement.clientHeight
      divElement.style.height = (predict + call + 190).toString() + "px";
    }
  }

  window.setInterval(function logname() {
    adjustheight(id)
  }, 500);
}

autoheight("main");

//连接websocket
let url = "ws://localhost:8000/websocket"
var socket = new WebSocket(url)

//定义断开websocket连接之后的回调函数
socket.onclose = function (evt) {
  console.log("Websocket连接已经关闭！");
  location.reload();
};

//删除成功提示
function delelte_success() {
  message.success('删除模型成功！')
}

//定义收到服务器消息之后如何处理
socket.onmessage = function (message) {
  //获取服务器返回的消息
  //输出消息，方便debug
  console.log('收到服务器消息：', message.data)
  //处理服务器消息为JSON
  var data = JSON.parse(message.data);
  //输出模式
  console.log(data.mode)

  //判断是否为模型列表
  if (data.mode === "model_list") {
    // 执行字符串
    model_list.value = eval(data.model_list);
    console.log(model_list.value)
  }else if (data.mode === 'message'){
    //模型运行日志
    // model_log.push(data.messages)
    // console.log(model_log)
    handleAdd_result(data.messages)
    //模型运行状态
    model_state.value = "loading"
  }else if (data.mode === 'success'){
    //模型运行日志
    // model_log.push("运行结果： 自变量X：" + data.x_number + "因变量Y：" + data.y_number)
    // console.log(model_log)
    handleAdd_result("运行结果： 自变量X：" + data.x_number + "，因变量Y：" + data.y_number)
    //模型运行状态
    model_state.value = "success"
  }else if (data.mode === 'model_delete_success'){
    delelte_success()
  }
}

//获取模型列表
function get_model_list() {
  socket.send(JSON.stringify({
    "mode": "model_list"
  }))
}

//删除模型
function delete_model(model_name) {
  socket.send(JSON.stringify({
    "mode": "model_delete",
    "model_name": model_name
  }))
}

//运行模型
function run_model() {
  if (mode.value === "predict") {
    socket.send(JSON.stringify({
      "mode": "predict",
      "predict_mode": predict_mode.value,
      "learning_rate": learning_rate.value,
      "num_epochs": num_epochs.value,
      "degree": degree.value,
      "train_data": dataSource.value,
      "test_x": dataSource_next.value,
      "save_model": save_model.value,
      "model_name": model_name.value,
    }))
    showDrawer()
  } else if (mode.value === "use") {
    socket.send(JSON.stringify({
      "mode": "model_call",
      "model_name": model_list.value[value.value],
      "test_x": dataSource_next.value,
    }))
    showDrawer()
  }
}

//循环检测是否需要获取模型列表
setInterval(function () {
  if (mode.value === "use") {
    get_model_list()
  }
}, 1000);
</script>

<template>
  <div style="background: #ececec; padding: 30px" id="main">
    <!-- 抽屉-->
    <a-drawer
        v-model:open="open"
        class="custom-class"
        root-class-name="root-class-name"
        title="模型运行结果"
        placement="right"
        @after-open-change="afterOpenChange"
    >
      <a-space v-if="model_state === 'loading'">
        <h1 style="color: #00a6ff;">模型运行中...</h1>
        <a-spin size="large"/>
      </a-space>
      <a-space v-if="model_state === 'success'">
        <h1 style="color: #00ff22;">模型运行完成！</h1>
        <CheckCircleFilled style="color: #00ff22; font-size: 35px;"/>
      </a-space>

      <a-table bordered :data-source="dataSource_result" :columns="columns_result">
        <template #bodyCell="{ column, text, record }">
        </template>
      </a-table>

    </a-drawer>
    <!-- 训练模型卡片 -->
    <a-card title="训练模型" :bordered="false" style="width: 500px;margin: auto;" id="predict">
      <a-radio-group v-model:value="mode" button-style="solid">
        <a-radio-button value="predict">训练新的模型</a-radio-button>
        <a-radio-button value="use">使用已有模型</a-radio-button>
      </a-radio-group>
      <br><br>

      <a-space :size="[8, 16]" wrap v-if="mode === 'use'">
        <a-radio-group v-model:value="value" v-for="(item, index) in model_list" v-if="mode === 'use'">
          <a-radio :style="radioStyle" :value="index">{{ item }}</a-radio>
          <a-button type="primary" @click="delete_model(item)">删除模型</a-button>
        </a-radio-group>
      </a-space>

      <h3 v-if="mode === 'predict'">选择模型</h3>
      <h5 v-if="mode === 'predict'">*线性回归模型适用于因变量和自变量之间存在线性关系的情况</h5>
      <h5 v-if="mode === 'predict'">*多项式回归模型适用于因变量和自变量之间存在非线性关系的情况</h5>
      <a-radio-group v-model:value="predict_mode" v-if="mode === 'predict'" button-style="solid">
        <a-radio-button value="LinearRegression">线性回归模型</a-radio-button>
        <a-radio-button value="PolynomialRegression">多项式回归模型</a-radio-button>
      </a-radio-group>
      <br v-if="mode === 'predict'"><br v-if="mode === 'predict'">

      <h3 v-if="mode === 'predict'">设置参数</h3>
      <a-input-number id="inputNumber" v-model:value="learning_rate" :step="0.01" addon-before="学习率"
                      v-if="mode === 'predict'"/>
      <a-input-number id="inputNumber" v-model:value="num_epochs" :min="1" addon-before="训练轮数"
                      v-if="mode === 'predict'"/>
      <a-input-number id="inputNumber" v-model:value="degree" :min="1" addon-before="多项式的次数"
                      v-if="mode === 'predict' && predict_mode === 'PolynomialRegression'"/>
      <br v-if="mode === 'predict'"><br v-if="mode === 'predict'">

      <h3 v-if="mode === 'predict'">设置训练数据</h3>
      <a-button class="editable-add-btn" style="margin-bottom: 8px" @click="handleAdd" v-if="mode === 'predict'">
        添加数据
      </a-button>
      <a-table bordered :data-source="dataSource" :columns="columns" v-if="mode === 'predict'">
        <template #bodyCell="{ column, text, record }">
          <template v-if="column.dataIndex === 'x'">
            <div class="editable-cell">
              <div v-if="editableData[record.key]" class="editable-cell-input-wrapper">
                <a-input v-model:value="editableData[record.key].x" @pressEnter="save(record.key)"/>
                <check-outlined class="editable-cell-icon-check" @click="save(record.key)"/>
              </div>
              <div v-else class="editable-cell-text-wrapper">
                {{ text || ' ' }}
                <edit-outlined class="editable-cell-icon" @click="edit(record.key)"/>
              </div>
            </div>
          </template>
          <template v-if="column.dataIndex === 'y'">
            <div class="editable-cell">
              <div v-if="editableData[record.key]" class="editable-cell-input-wrapper">
                <a-input v-model:value="editableData[record.key].y" @pressEnter="save(record.key)"/>
                <check-outlined class="editable-cell-icon-check" @click="save(record.key)"/>
              </div>
              <div v-else class="editable-cell-text-wrapper">
                {{ text || ' ' }}
                <edit-outlined class="editable-cell-icon" @click="edit(record.key)"/>
              </div>
            </div>
          </template>
          <template v-else-if="column.dataIndex === 'operation'">
            <a-popconfirm
                v-if="dataSource.length"
                title="确定删除？"
                @confirm="onDelete(record.key)"
            >
              <a>删除数据</a>
            </a-popconfirm>
          </template>
        </template>
      </a-table>
    </a-card>
    <br>
    <!-- 调用模型卡片 -->
    <a-card title="调用模型" :bordered="false" style="width: 500px;margin: auto;" id="call">
      <h3>设置需要预测值的自变量X</h3>

      <a-button class="editable-add-btn" style="margin-bottom: 8px" @click="handleAdd_next">
        添加数据
      </a-button>
      <a-table bordered :data-source="dataSource_next" :columns="columns_next">
        <template #bodyCell="{ column, text, record }">
          <template v-if="column.dataIndex === 'x'">
            <div class="editable-cell">
              <div v-if="editableData_next[record.key]" class="editable-cell-input-wrapper">
                <a-input v-model:value="editableData_next[record.key].x" @pressEnter="save_next(record.key)"/>
                <check-outlined class="editable-cell-icon-check" @click="save_next(record.key)"/>
              </div>
              <div v-else class="editable-cell-text-wrapper">
                {{ text || ' ' }}
                <edit-outlined class="editable-cell-icon" @click="edit_next(record.key)"/>
              </div>
            </div>
          </template>
          <template v-else-if="column.dataIndex === 'operation'">
            <a-popconfirm
                v-if="dataSource_next.length"
                title="确定删除？"
                @confirm="onDelete_next(record.key)"
            >
              <a>删除数据</a>
            </a-popconfirm>
          </template>
        </template>
      </a-table>

    </a-card>
    <br>
    <!--保存模型卡片 -->
    <a-card title="保存模型" :bordered="false" style="width: 500px;margin: auto;" v-if="mode === 'predict'" id="save">
      <a-radio-group v-model:value="save_model" button-style="solid">
        <a-radio-button value="true">保存模型</a-radio-button>
        <a-radio-button value="F">不保存模型</a-radio-button>
      </a-radio-group>
      <br><br v-if="save_model === 'true'">
      <h3 v-if="save_model === 'true'">请设置模型名称，只能使用英文，不能存在中文或字符数字</h3>
      <a-input v-model:value="model_name" addon-after=".pth" v-if="save_model === 'true'"/>
    </a-card>
    <br><br>
    <div style="text-align: center;">
      <a-button type="primary" style="margin: auto;" size="large" @click="run_model()">
        <template #icon>
          <PoweroffOutlined/>
        </template>
        运行模型！
      </a-button>
    </div>
    <br><br>
  </div>
</template>

<style lang="less" scoped>
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
</style>
