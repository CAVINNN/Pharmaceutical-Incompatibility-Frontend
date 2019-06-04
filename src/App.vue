<template>
  <div id="app" v-loading.fullscreen.lock="fullscreenLoading">
    <!--header-->
    <el-row>
      <el-col :span="24">
        <div class="header"><img src="./assets/img/logo.png" />中西药配伍禁忌系统</div>
      </el-col>
    </el-row>

    <!--content-->
    <el-row>
      <el-col :span="10">
        <div class="content-left" :style="{ height: contentHeight + 'px' }">
          <div class="content-left-header">中药</div>
          <div class="content-left-content">
            <template v-for="cmCount in cmCounts">
              <div class="chineseMedicines">
                <el-select v-model="cmCount.val" filterable placeholder="请选择">
                  <el-option
                    v-for="item in chineseMedicines"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
                </el-select>
                <el-button type="primary" icon="el-icon-circle-close-outline" @click="deleteCM(cmCount.index)"></el-button>
              </div>
            </template>
            <div class="left-add">
              <el-button type="primary" @click="addCM" style="font-family: 'PingFang SC'">添加中药</el-button>
            </div>
          </div>
        </div>
      </el-col>
      <el-col :span="4">
        <div class="middle" :style="{ height: contentHeight + 'px' }">
          <!--<img src="./assets/img/arrow4.png" />-->
          <el-button type="primary" class="confirmButton" :style="{ margin: confirmButtonHeight+'px 0px 0px 17.5%' }" @click="mix">药物配伍</el-button>
        </div>
      </el-col>
      <el-col :span="10">
        <div class="content-right" :style="{ height: contentHeight + 'px' }">
          <div class="content-right-header">西药</div>
          <div class="content-right-content">
            <template v-for="wmCount in wmCounts">
              <div class="chineseMedicines">
                <el-select v-model="wmCount.val" filterable placeholder="请选择">
                  <el-option
                    v-for="item in westernMedicines"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
                </el-select>
                <el-button type="primary" icon="el-icon-circle-close-outline" @click="deleteWM(wmCount.index)"></el-button>
              </div>
            </template>
            <div class="right-add">
              <el-button type="primary" @click="addWM" style="font-family: 'PingFang SC'">添加西药</el-button>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>

  </div>
</template>

<script>
export default {
  name: 'app',
  mounted () {
      this.fullscreenLoading = true;
      this.getContentHeight();
      this.getInitMedicinesData();
      this.fullscreenLoading = false;
  },

  data () {
    return {
      fullscreenLoading: false,
      contentHeight: 0,
      confirmButtonHeight: 0,
      chineseMedicines: [],
      westernMedicines: [],
      cmCounts: [],
      wmCounts: [],
    }
  },

  methods: {
      getContentHeight() {
          this.contentHeight = (document.body.clientHeight-60-30-10-40);
          this.confirmButtonHeight = (this.contentHeight/2.3);
      },

      getInitMedicinesData() {
        let self = this;

        // Ajax获取中药实例
        this.$ajax.post('http://localhost:8080/PI/getChineseMedicines', {},
        {
          headers: {'Content-Type': 'application/json'}
        }
      ).then(function(res){
          let medicines = res.data;
          for( let i=0; i<medicines.length; i++ ){
              let obj = new Object();
              obj.value = medicines[i];
              obj.label = medicines[i];
              self.chineseMedicines.push(obj);
          }
        })
        .catch(function(err){
          window.alert("获取中药实例出错！");
        });

        // Ajax获取西药实例
        this.$ajax.post('http://localhost:8080/PI/getWesternMedicines', {},
          {
            headers: {'Content-Type': 'application/json'}
          }
        ).then(function(res){
            let westernMedicines = res.data;
            for( let i=0; i<westernMedicines.length; i++ ){
              let obj = new Object();
              obj.value = westernMedicines[i];
              obj.label = westernMedicines[i];
              self.westernMedicines.push(obj);
            }
        })
          .catch(function(err){
            window.alert("获取西药实例出错！");
          });

      },

      addCM() {
          let obj = new Object();
          obj.index = this.cmCounts.length;
          obj.val = "";
          this.cmCounts.push(obj);
      },

      deleteCM(index) {
        if( this.cmCounts.length != 0 ){
            this.cmCounts.splice(index, 1);
            for(let i=index; i<this.cmCounts.length; i++){
                this.cmCounts[i].index--;
            }
        }
      },

      addWM() {
          let obj = new Object();
          obj.index = this.wmCounts.length;
          obj.val = "";
          this.wmCounts.push(obj);
      },

      deleteWM(index) {
        if( this.wmCounts.length != 0 ){
          this.wmCounts.splice(index, 1);
          for(let i=index; i<this.wmCounts.length; i++){
            this.wmCounts[i].index--;
          }
        }
      },

      mix() {
          let self = this;
          let CM = [];
          let WM = [];
          for( let i=0; i<this.cmCounts.length; i++ ){
              CM.push( this.cmCounts[i].val );
          }
          for( let i=0; i<this.wmCounts.length; i++ ){
              WM.push( this.wmCounts[i].val );
          }

          if( CM.length == 0 ){
              this.$message({
                showClose: true,
                message: '中药栏不得为空！',
                type: 'error',
                center: true
              });
              return ;
          }

        if( WM.length == 0 ){
          this.$message({
            showClose: true,
            message: '西药栏不得为空！',
            type: 'error',
            center: true
          });
          return ;
        }

        this.$ajax.post('http://localhost:8080/PI/doMix', {
            chineseMedicines: CM,
            westernMedicines: WM
          },
          {
            headers: {'Content-Type': 'application/json'}
          }
        ).then(function(res){
            console.log(res.data);
            let backData = res.data;
            if( res.data.length != 0 ){
              let str = "";
              console.log("here");
              for( let i=0; i<backData.length; i++ ){
                  let chinese = backData[i].slice(0, backData[i].indexOf("vs"));
                  let western = backData[i].slice(backData[i].lastIndexOf("vs")+2);
                  str = str + chinese + " --> 禁忌 --> " + western + "<br />";
              }
              self.$message({
                showClose: true,
                message: '药物之间拥有配伍禁忌！',
                type: 'warning',
                center: true,
                onClose: function () {
                  self.$alert(str, '具体药物禁忌', {
                    dangerouslyUseHTMLString: true,
                    confirmButtonText: '确定'
                  });
                }
              });
            }
            else{
              self.$message({
                showClose: true,
                message: '药物之间无配伍禁忌！',
                type: 'success',
                center: true
              });
            }
        })
          .catch(function(err){
            window.alert("配伍出错！");
          });

      }
  }
}
</script>

<style>
  html, body{
    margin: 0px;
    height: 100%;
    background: #f5f7f9;
    font-family: "PingFang SC";
  }

  .header{
    /*background: #fff;*/
    color: #fff;
    background:#464c5b;
    height: 60px;
    font-size: 24px;
    box-shadow: 0 1px 1px rgba(0, 0, 0, .1);
  }

  .header img{
    vertical-align: -50%;
    width: 50px;
    height: 50px;
    margin: 6px 5px 0px 30px;
  }

  .middle{
    margin: 30px 10px 0px 10px;
  }

  .confirmButton{
    float: left;
    font-family: "Microsoft YaHei";
    font-size: 150%;
    /*margin: 0px 0px 0px 18%;*/
  }

  .content-left{
    margin: 30px 0px 10px 30px;
    overflow: auto;
    background: #fff;
    /*border: 3px groove gainsboro;*/
    box-shadow: 5px 5px 5px 3px rgba(0, 0, 0, .1);
    border-radius: 4px;
  }

  .content-left-header{
    width: 80%;
    margin: 0 auto 10px auto;
    padding: 10px 0 10px 0;
    border-bottom: 1.5px solid #464c5b;
    text-align: center;
    font-size: 22px;
  }

  .content-left-content{
    width: 80%;
    margin: 0 auto;
    overflow: auto;
  }

  .chineseMedicines{
    text-align: center;
    margin: 10px 0 0 0;
  }

  .left-add{
    width: 100%;
    text-align: center;
    margin: 10px 0 10px 0;
  }

  .content-right-header{
    width: 80%;
    margin: 0 auto 20px auto;
    padding: 10px 0 10px 0;
    border-bottom: 1.5px solid #464c5b;
    text-align: center;
    font-size: 22px;
  }

  .content-right-content{
    width: 80%;
    margin: 0 auto;
  }

  .content-right{
    margin: 30px 30px 10px 0px;
    overflow: hidden;
    background: #fff;
    /*border: 3px groove gainsboro;*/
    box-shadow: 5px 5px 5px 3px rgba(0, 0, 0, .1);
    border-radius: 4px;
  }

  .right-add{
    width: 100%;
    text-align: center;
    margin: 10px 0 10px 0;
  }

  <!-- element css -->

  .el-input--suffix .el-input__inner{
    font-family: "PingFang SC";
  }

</style>
