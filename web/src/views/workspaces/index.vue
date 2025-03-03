<template>
  <div class="dashboard-container">
    <div ref="code" class="code">
      <el-row :gutter="20" class="pt15 px5" style="height: 100%">
        <el-col :span="12" style="height: 100%">
          <div class="checkbox-container">
            <div class="checkbox-header">
              <el-checkbox
                v-model="checkAllDevice"
                :indeterminate="isDevicesIndeterminate"
                @change="handleDeviceCheckAllChange"
              >设备列表</el-checkbox>
            </div>
            <div class="checkbox-group">
              <el-checkbox-group v-model="checkedDevices" @change="handleCheckedDeviceChange">
                <div v-for="item in devices" :key="item.device_id" class="checkbox-item">
                  <el-checkbox
                    :label="item.device_id"
                  >{{ item.name }} {{ item.is_online ? '' : `（离线）` }}</el-checkbox>
                </div>
              </el-checkbox-group>
            </div>
          </div>
        </el-col>
        <el-col :span="12" style="height: 100%">
          <div class="checkbox-container">
            <div class="checkbox-header">
              <span style="color: #606266; font-size: 14px; font-weight: 500;">脚本列表</span>
            </div>
            <div v-loading="listLoading" class="checkbox-group">
              <div v-for="item in scripts" :key="item.script_id" class="checkbox-item">
                <el-radio v-model="checkedScript" :label="item.script_id">{{ item.script_name }}</el-radio>
              </div>
            </div>
          </div>
        </el-col>
      </el-row>
    </div>
    <div ref="divide" class="divide" />
    <device-log class="device_log">
      <div>
        <el-button plain round size="mini" @click="stopAllScript">Stop All</el-button>
        <el-button plain round size="mini" @click="runScript">Run Script</el-button>
      </div>
    </device-log>
  </div>
</template>

<script>
import request from "@/utils/request";
import DeviceLog from "@/components/DeviceLog";
import { mapGetters } from "vuex";
export default {
  name: "Dashboard",
  components: {
    DeviceLog
  },
  data() {
    return {
      checkAllDevice: false,
      checkedDevices: [],
      isDevicesIndeterminate: true,
      checkAllScript: false,
      checkedScript: null,
      isScriptsIndeterminate: true,
      scripts: []
    };
  },
  computed: {
    ...mapGetters(["name"]),
    devices() {
      return this.$store.state.device.list;
    }
  },
  created() {
    this.getScripts();
  },
  mounted() {
    this.dragDivide();
  },
  methods: {
    getScripts() {
      this.listLoading = true;
      request({
        url: "/script/get_script_list",
        method: "get",
        params: {}
      }).then(res => {
        this.scripts = res.data.scripts;
        this.listLoading = false;
      });
    },
    handleDeviceCheckAllChange(val) {
      console.log(val);
      this.checkedDevices = val ? this.devices.map(i => i.device_id) : [];
      this.isDevicesIndeterminate = false;
    },
    handleCheckedDeviceChange(value) {
      const checkedCount = value.length;
      this.checkAllDevice = checkedCount === this.devices.length;
      this.isDevicesIndeterminate =
        checkedCount > 0 && checkedCount < this.devices.length;
    },
    handleCheckedScriptChange(value) {
      console.log(value);
      const checkedCount = value.length;
      this.checkAllScript = checkedCount === this.scripts.length;
      this.isScriptsIndeterminate =
        checkedCount > 0 && checkedCount < this.scripts.length;
    },
    runScript() {
      request
        .post("/script/run2", {
          script_id: this.checkedScript,
          devices: this.checkedDevices
        })
        .then(res => {
          this.$message({
            message: "操作成功！",
            type: "success"
          });
        })
        .finally(() => {});
    },
    stopAllScript() {
      request
        .post("/script/stop_all", {
          devices: this.checkedDevices
        })
        .then(res => {
          this.$message({
            message: "操作成功！",
            type: "success"
          });
        })
        .finally(() => {});
    },
    dragDivide() {
      const codeMirrorWrapEle = this.$refs.code;
      const divideRef = this.$refs.divide;
      // var mouseDownX;
      var mouseDownY;
      // var initX;
      var initY;
      var doc_onmousemove;
      var doc_onmouseup;
      var flag = false;
      var isMoving = false;

      divideRef.onmousedown = function(e) {
        var obj = divideRef; // 移动目标元素
        // 表示鼠标已按下
        flag = true;

        // 鼠标按下时的鼠标所在的X，Y坐标
        // mouseDownX = e.clientX;
        mouseDownY = e.clientY;

        // 初始位置的X，Y 坐标
        // initX = obj.offsetLeft;
        initY = obj.offsetTop;

        // 保存原来绑定在document的事件
        doc_onmousemove = document.onmousemove;
        doc_onmouseup = document.onmouseup;

        document.onmousemove = move;
        document.onmouseup = obj.onMouseOut = stop;

        function move(e) {
          if (flag && !isMoving) {
            // obj.style.left =
            //   parseInt(e.clientX) - parseInt(mouseDownX) + parseInt(initX) + "px";
            codeMirrorWrapEle.style.height =
              parseInt(e.clientY) -
              parseInt(mouseDownY) +
              parseInt(initY) +
              "px";

            console.log(codeMirrorWrapEle.style.height);

            window.dispatchEvent(new Event("resize"));
            isMoving = true;
            setTimeout(function() {
              isMoving = false;
            }, 10);
          }
          return false;
        }
        function stop() {
          flag = false;
          document.onmousemove = doc_onmousemove; // 原来的事件回复绑定
          document.onmouseup = doc_onmouseup;
        }
        return false; // 可以防止在拖动的时候选中文本
      };
    }
  }
};
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 0px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
.checkbox-container {
  border: 1px solid #ebeef5;
  border-radius: 4px;
  overflow: hidden;
  height: 100%;
}
.checkbox-header {
  height: 40px;
  line-height: 40px;
  background: #f5f7fa;
  margin: 0;
  padding-left: 15px;
  border-bottom: 1px solid #ebeef5;
  box-sizing: border-box;
  color: #000;
}
.checkbox-group {
  padding: 5px 15px 15px 15px;
  height: 100%;
  overflow: auto;
}
.checkbox-item {
  margin-top: 10px;
  display: block;
}
.code {
  height: 300px;
  overflow: auto;
  overflow-x: hidden;
}
.divide {
  font-size: 13px;
  padding: 6px 0 6px 10px;
  background: #fff;
  color: #303133;
  cursor: ns-resize;
  -moz-user-select: none; /*火狐*/
  -webkit-user-select: none; /*webkit浏览器*/
  -ms-user-select: none; /*IE10*/
  -khtml-user-select: none; /*早期浏览器*/
  user-select: none;
}
</style>
