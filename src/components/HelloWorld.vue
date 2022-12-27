<script setup lang="ts">
import { ref, onMounted } from "vue";

const checked = ref(false);
const onChange = (index: number) => {
  checkList.value[index] = !checkList.value[index];
};

var NameList = ref([]);
var checkList = ref([] as any);
var todayChance = ref([] as any);
var winner = ref("");

const LoadStorage = async () => {
  NameList.value = JSON.parse(localStorage.getItem("dataJson") as string);
  console.log("用户数据", NameList.value);
};

function SetStorage() {
  var array = [
    { name: "张国志[5330]", join: 1, win: 0 },
    { name: "陈兆年[6663]", join: 1, win: 0 },
    { name: "陈文博[0090]", join: 1, win: 0 },
    { name: "yyyyy[4267]", join: 1, win: 0 },
    { name: "王龙[6221]", join: 1, win: 0 },
    { name: "申庆涛[9627]", join: 1, win: 0 },
    { name: "管坤[8167]", join: 1, win: 0 },
    { name: "张超[5760]", join: 1, win: 0 },
  ];
  localStorage.setItem("dataJson", JSON.stringify(array));

  LoadStorage();
}

function RandomPerson() {
  todayChance.value = [];
  var JoinList: any = [];
  if (!NameList.value || NameList.value.length == 0) {
    alert("没有参与人");
    return;
  }
  for (var i = 0; i < NameList.value.length; i++) {
    if (checkList.value[i]) {
      JoinList.push(NameList.value[i]);
    }
  }

  if (JoinList.length == 0) {
    alert("至少选择一个");
    return;
  }

  var n = JoinList.length;
  for (var i = n - 1; i >= 0; i--) {
    var random = Math.round(Math.random() * i);
    var tmp = JoinList[i];
    JoinList[i] = JoinList[random];
    JoinList[random] = tmp;
  }
  console.log(JoinList, "新队列");
  winner.value = JoinList[0].name;
  // swap(arr[i], arr[rand() % (i + 1)])
}

function weighted_random(options: any) {
  var i;
  var weights: any = [];
  for (i = 0; i < options.length; i++)
    weights[i] = options[i].weight + (weights[i - 1] || 0);
  var random = Math.random() * weights[weights.length - 1];
  for (i = 0; i < weights.length; i++) if (weights[i] > random) break;
  return options[i].name;
}

function getPerson() {
  LoadStorage();
  todayChance.value = [];
  var JoinList: any = [];
  if (!NameList.value || NameList.value.length == 0) {
    alert("没有参与人");
    return;
  }
  for (var i = 0; i < NameList.value.length; i++) {
    if (checkList.value[i]) {
      JoinList.push(NameList.value[i]);
    }
  }

  if (JoinList.length == 0) {
    alert("至少选择一个");
    return;
  }

  for (var i = 0; i < JoinList.length; i++) {
    var aaa = {
      name: JoinList[i].name,
      weight: (JoinList[i].join - JoinList[i].win) / JoinList[i].join,
    };
    todayChance.value.push(aaa);
  }

  winner.value = weighted_random(todayChance.value);

  WriteRecord(winner.value);
}
function WriteRecord(name: string) {
  for (var i = 0; i < NameList.value.length; i++) {
    if (checkList.value[i]) {
      NameList.value[i]["join"]++;
      if (NameList.value[i]["name"] == name) {
        NameList.value[i]["win"]++;
      }
    }
  }
  localStorage.setItem("dataJson", JSON.stringify(NameList.value));
}

function reset() {
  localStorage.removeItem("dataJson");
  SetStorage();
}

onMounted(() => {
  LoadStorage();
  if (NameList.value) {
    for (var i = 0; i < NameList.value.length; i++) {
      checkList.value.push(true);
    }
  }
});
</script>

<template>
  <div class="main">
    <p>
      <el-tag class="tag_top"
              type="success">{{winner}}</el-tag>
    </p>
    <el-check-tag v-for="(item,index) in NameList"
                  :key="index"
                  :checked="checkList[index]"
                  @change="onChange(index)"
                  class="el-check-tag">{{item['name']}}</el-check-tag>
    <p>
      <el-button class="el-button-go"
                 type="danger"
                 @click="RandomPerson()">摇一摇</el-button>
    </p>
    <!-- <div class="table">
      <el-table :data="todayChance"
                stripe
                style="width: 100%">
        <el-table-column prop="name"
                         label="Name"
                         width="180" />
        <el-table-column label="权重">
          <template #default="scope">
            <div style="display: flex; align-items: center">
              <span style="margin-left: 10px">{{ (scope.row.weight*100).toFixed(2) }}</span>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div> -->

    <el-button type="danger"
               @click="reset()"
               link>恢复初始数据</el-button>
  </div>
</template>

<style scoped>
.main {
  width: 960px;
}
.tag_top {
  margin: 8px;
  width: 250px;
  height: 100px;
  align-items: center;
  line-height: 100px;
  font-size: 40px;
  border-radius: 20px;
}

.table {
  width: 500px;
  margin: 0 auto;
}

.el-check-tag {
  margin: 8px;
  width: 250px;
  height: 100px;
  align-items: center;
  line-height: 100px;
  font-size: 40px;
  border-radius: 20px;
}

.el-button-go {
  font-size: 40px;
  width: 200px;
  height: 100px;
}
</style>
