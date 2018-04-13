<template>
  <div class="caseList">
    <div class="cityDialog hide">
      <div class="cityDialogBox">
        <span @click="closeCityDiolag" class="closeCityDiolag">×</span>
        <div class="cityNameBox clear">
          <span class="citySimpleName" v-for="item in cityData" @click="selectCity(item)">{{item}}</span>
        </div>
      </div>
    </div>
    <div class="creatCaseDialog hide">
      <div class="creatCaseDialogBox">
        <span @click="closCreatDiolag" class="closCreatDiolag">×</span>
        <div class="oneMonitor clear">
          <h4 class="dialogTitle">创建案件</h4>
          <div class="clear scrollBox">
            <div style="margin-top:20px;">
              <div class="addinsitituteInput">
                <span class="addinsitituteSpan">报案人手机号</span>
                <input type="tel" class="creatInput" v-model="phoneno"  maxlength="11" placeholder="请输入报案人手机号"/>
              </div>
              <div class="addinsitituteInput">
                <span class="addinsitituteSpan">报案人车牌号</span>
                <input type="text" @click="openCityDialog" class="creatInputNo"  readonly :value="getCity" />
                <input class="creatInput" type="text" v-model="licensenoTwo" @keyup="upcase()" style="margin-left:-6px;width:165px;" placeholder="请输入报案人车牌号"/>
              </div>
              <div class="addinsitituteInput">
                <span class="addinsitituteSpan">报案人姓名</span>
                <input class="creatInput" v-model="person" type="text" placeholder="请输入报案人姓名"/>
              </div>

              <div class="addinsitituteInput">
                <span class="addinsitituteSpan">工号号</span>
                <input class="creatInput"  v-model="reportno" type="text" placeholder="请输入工号号"/>
              </div>
              <div class="addinsitituteInput" style="margin-left:120px; margin-top: 25px;">
                <span class="addinsitituteSure backColorGreen" @click="creatNewCase">确定</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div style="margin-top: 20px;">
      <div class="creatCase">
        <span class="right creatCaseSpan backColorGreen" @click="openCreatCase">创建案件</span></div>
      <div class="caseListTable" v-if="tableActive">
        <!--<div class="tableTitle">-->
          <!--<span>共: {{pages}}页,</span>-->
          <!--<span>{{totalCount}}条, </span>-->
          <!--<span>当前页: {{currentCount}}条</span>-->
        <!--</div>-->
        <table class="table" border="0" cellspacing="0" cellpadding="0" style="border-top: 1px solid #bbb;margin:0 auto;margin-top: 45px;">
          <thead>
          <tr>
            <th style="border-left:1px solid #bbb;">
              编号
            </th>
            <th>
              报案人车牌号
            </th>
            <th>
              报案人姓名
            </th>
            <th>
              报案人手机号
            </th>
            <th style="width:12%;">
             创建时间
            </th>
            <th>
              工号
            </th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(item,index) in tableData">
            <td>{{index+1}}</td>
            <td>{{item.reporterPlateNumber}}</td>
            <td>{{item.reporterName}}</td>
            <td>{{item.reporterPhone}}</td>
            <td>{{item.createTime}}</td>
            <td style="width:160px;">{{item.jobNumber}}</td>

          </tr>
          </tbody>
        </table>
        <!--<div>-->
          <!--<el-pagination  @current-change="handleCurrentChange"-->
                          <!--:current-page="currentPageNo"-->
                          <!--:page-size = "pageSize"-->
                          <!--layout="prev,pager,next"-->
                          <!--:total="totalCount">-->
          <!--</el-pagination>-->
        <!--</div>-->
      </div>
      <div class="caseListTable" v-else>
        <p style="text-align:center;margin-top: 15px;">暂无数据</p>
      </div>
      <div class="footer">
        <div class="footer-inner footer_p">
          <p> 版权所有：北京中车宝联科技有限责任公司</p>
        </div>
      </div>
    </div>
    <case-detail v-if="caseDetailActive"></case-detail>
    <sign-Seats v-if="signSeatsActive"></sign-Seats>
  </div>
</template>
<script>
  import axios from 'axios'
  import caseDetail from '../components/caseDetail'
  import signSeats from '../components/signSeats'
  export default {
    data() {
      return {
        userId: '',
        cityData: ['京','津','冀','晋','蒙','辽','吉','黑','沪','苏','浙','皖','闽','赣','鲁','豫','鄂','湘','粤','贵','云','藏','陕','甘','青','宁','新','琼','渝','川','桂'],
        reportno: '',
        getCity: "京",
        phoneno: '',
        licensenoTwo: '',
        person: '',
        tableActive: false,
        reporterPhoneNo: "",
        reporterCarLicenseNo: "",
        reportInsuranceNo: "",
        orgCode: '',
        accidentStartTime:"",
        accidentEndTime: "",
        handleStartTime: "",
        handleEndTime: "",
        insuranceCompanyCode: '',
        surveyStatus: '',
        pageSize: 10,
        pages: '',

        currentPageNo: 1,
        currentCount: "",
        survey: "",
        tableData: [],
        totalCount: 0,
        companeyOptions: {},
        organizations: [],
        value6: '',
        value7: '',
        showCaseDetail: false,
        caseDetailActive: false,
        signSeatsActive: false,
        caseListActive: false,
        time: '',
      }
    },
    watch: {
      getUserIcons(val) {
        this.caseDetailActive = val;
      },
      getsignSeatsActive(val) {
        this.signSeatsActive = val;
      },
      getCaseListActive(val) {
        this.caseListActive = val;

        if(this.caseListActive){
          this.getCaseList()
        }
      },
    },

    created() {
      this.userId = localStorage.getItem('userId');
      this.getCaseList();
    },
    mounted() {
      this.caseDetailActive = this.$store.state.caseDetailActive;
    },
      methods: {
        creatNewCase(){
          this.licensenoTwo = this.licensenoTwo.replace(/\s|\xA0/g,"");
          this.phoneno = this.phoneno.replace(/\s/g,"");
          var reg = new RegExp("^[0-9]*$");
          if(!reg.test(this.phoneno)){
            this.open4("请输入正确手机号")
          }else if(this.phoneno.length<11){
            this.open4("请输入正确手机号")
          }else if(this.phoneno == ""){
            this.open4("请输入手机号")
          }else if(this.licensenoTwo == ""){
            this.open4("请输入车牌号")
          }else if(this.person == ""){
            this.open4("请输入报案人姓名")
          }else if(this.reportno == ''){
            this.open4("请输入工号")
          }else{
            var paramData = {
              "userId": this.userId,
              'reporterPhone': this.phoneno,
              'reporterPlateNumber':this.getCity+this.licensenoTwo,
              'reporterName': this.person,
              'jobNumber': this.reportno
            };
            axios.post(this.ajaxUrl+"/duty/v1/create",paramData)
              .then(response => {
                if(response.data.rescode == 200){

                  this.open2(response.data.resdes)
                  $(".creatCaseDialog").addClass("hide")
                  this.getCaseList()
                }else{
                  if(response.data.rescode == "300"){
                    this.$router.push({path:"/"})
                  }
                  this.open4(response.data.resdes);
                }
              }, err => {
                console.log(err);
              })
              .catch((error) => {
                console.log(error)
              })
          }
        },
        upcase(){
          this.licensenoTwo = this.licensenoTwo.toUpperCase();
        },
        openCityDialog(){//打开城市
          $(".cityDialog").removeClass("hide")
        },
        closeCityDiolag(){//关闭城市遮盖层
          $(".cityDialog").addClass("hide")
        },
        selectCity(city){
          this.getCity = city;
          $(".cityDialog").addClass("hide")
        },
        closCreatDiolag(){
          $(".creatCaseDialog").addClass('hide')
        },
        openCreatCase(){//打开创建案件
          $(".creatCaseDialog").removeClass('hide');
        },
        open4(resdes) {
          this.$message.error(resdes);
        },
        open2(resdes) {
          this.$message.success(resdes);
        },
        signSeats(id){
          this.$store.commit('getsurveyOrderId',id);
          axios.get(this.ajaxUrl+'/web-surveyor/v1/list')
            .then(response => {
              if(response.data.rescode == 200){
                if(response.data.data.length != 0){
                  localStorage.setItem("signSeatData",JSON.stringify(response.data.data))
                  this.$store.commit('setSignSeatsActive', true);
                  this.signSeatsActive = this.$store.state.signSeatsActive;
                }else{
                  this.open2("暂无数据")
                }
              }else{
                this.open4(response.data.resdes)
                if(response.data.rescode == 300){
                  this.$router.push({path:'/'})
                }
              }
            }, err => {
              console.log(err);
            })
            .catch((error) => {
              console.log(error)
            })
        },
        getCaseList() {
          var userId = this.userId;
          axios.get(this.ajaxUrl+"/duty/v1/new/duty/list/"+userId)
            .then(response => {
              console.log(this.tableData)
              if(response.data.rescode == 200){
                this.tableData = response.data.data;
console.log(this.tableData)
                this.$store.commit('getcaseListActive', false)//监听调用列表接口关闭
                if(response.data.data.length !=0){
                  this.tableActive = true;
                  this.totalCount = parseInt(response.data.data.total);
                  this.currentCount = response.data.data.size;
                  this.pages = response.data.data.pages;
                }else{
                  this.tableActive = false;
                }
              }else{
                this.open4(response.data.resdes)
                if(response.data.rescode == 300){
                  this.$router.push({path:'/'})
                }
              }
            }, err => {
              console.log(err);
            })
            .catch((error) => {
              console.log(error)
            })
        },
        handleClick(row) {
          console.log(row);
        },
        formSure() {
          this.getCaseList()
        },
        resetData(){
          this.insuranceCompanyCode = "";
          this.orgCode = "";
          this.reporterCarLicenseNo = "";
          this.reporterPhoneNo = "";
          this.reportInsuranceNo = "";
          this.surveyStatus = "";
          this.accidentStartTime = "";
          this.accidentEndTime = "";
          this.handleStartTime = "";
          this.handleEndTime = "";
          this.value6 = "";
          this.value7 = "";
          this.getCaseList()
        },
        handleCurrentChange(currentPage) {//跳转
          //当前页改变调用接口  pageNo  pageSize
          this.currentPageNo = currentPage;
          this.getCaseList()
        },
        goCaseDetail(id,orderStatus){
            var paramData = {
              "id": parseInt(id),
              "orderStatus": orderStatus
//              "id": 455,
//              "orderStatus": 11
            }
            axios.post(this.ajaxUrl+"/survey-detail/v1/get",paramData)
              .then(response => {
                if(response.data.rescode == 200){
                  var data = JSON.stringify(response.data.data)
                  localStorage.setItem("caseDetailData",data);
                  this.$store.commit('setCaseDetailActive', true);
                  this.caseDetailActive = this.$store.state.caseDetailActive;
                }
              }, err => {
                console.log(err);
              })
              .catch((error) => {
                console.log(error)
              })
        }
      },
    components: {
      caseDetail,
      signSeats,
    },
    computed: {
     getUserIcons(){
       return this.$store.state.caseDetailActive;
     },
      getsignSeatsActive() {
        return this.$store.state.signSeatsActive;
      },
      getCaseListActive(){
        return this.$store.state.caseListActive;
      }
    },
    destroyed () {
      clearInterval(this.time);
    },
  }

</script>
<style scoped>
  .addinsitituteSure{
    text-align: center;
    color: #fff;
    font-size: 15px;
    padding: 10px 35px;

  }
  .footer{
    position: fixed;
    bottom: 0;
    left: 0;
    height: 60px;
    width: 100%;
    background: #2EAB3B;
  }
  .footer-inner{
    text-align: center;
    position: absolute;
    z-index: auto;
    left: 0;
    right: 0;
    bottom: 0;
  }
  .footer-inner p{
    line-height: 64px;
    font-size: 14px;
    color: #fff;
    margin: 0;
  }
  .cityDialog{
    background: rgba(0,0,0,0.3);
    width:100%;
    position: fixed;
    min-height: 100vh;
    top: 0;
    left: 0;
    z-index: 1000;
  }
  .cityDialogBox{
    width: 38%;
    margin: 6vh auto;
    background: #fff;
    padding: 20px;
    position: relative;
  }
  .closeCityDiolag{
    font-size: 42px;
    right: 15px;
    top: 0;
    position: absolute;
  }
  .cityDialogBox .cityNameBox{
    width: 85%;
    text-align: center;
    clear: both;
    margin-left: 30px;
  }
  .cityDialogBox .cityNameBox .citySimpleName{
    width: 10%;
    height: 35px;
    line-height: 35px;
    border: 1px solid #bbb;
    color: #232323;
    border-radius: 5px;
    text-align: center;
    margin-right: 10px;
    float: left;
    margin-top: 20px;
    cursor: pointer;
  }
  .addinsitituteInput{
    padding: 10px 0 5px 10%;
  }
  .addinsitituteInput .addinsitituteSpan{
    display: inline-block;
    min-width: 20%;
    font-size: 15px;
  }
  .addinsitituteInput .creatInput{
    height:35px;
    line-height:35px;
    padding-left: 6px;
    border: 1px solid #bbb;
    border-radius:4px;
    width: 200px;
  }
  .addinsitituteInput .creatInputNo{
    height:35px;
    line-height: 35px;
    border: 1px solid #bbb;
    border-radius:4px;
    width: 35px;
    background: #fff;
    border-right: none;
    text-align: center;
  }
  .creatCaseDialog{
    background: rgba(0,0,0,0.3);
    width:100%;
    position: fixed;
    min-height: 100vh;
    top: 0;
    left: 0;
    z-index: 100;
  }
  .creatCaseDialogBox{
    width: 30%;
    margin: 8vh auto;
    background: #fff;
    padding: 20px;
    position: relative;
    min-height: 500px;
  }
  .creatCaseSpan{
    margin-right:50px;
    padding: 10px 18px;
    color: #fff;
  }
  .creatCaseDialog .scrollBox{
    max-height: 60vh;
  }
  .closCreatDiolag{
    font-size: 42px;
    right: 15px;
    top: 0;
    position: absolute;
  }
.caseList{
  width: 82%;
  background: #fff;
  font-size: 15px;
  margin: 0 auto;
}

.caseListHeader form{
  width: 100%;
  margin-left: 25px;

  padding-bottom: 10px;
 }
  .caseListHeader input, .caseListHeader select{
    width: 11%;
    background: #fff;
    border: 1px solid #bbb;
    height: 35px;
    margin: 10px 30px 2px 10px;
    padding-left: 5px;
    border-radius: 4px;
    color: #232323;
  }
.caseListHeader select{
  border-radius: 6px;
}
.caseListHeader label{
  margin-left: 10px;
}
  .caseListSure, .caseListReset{
    border: 1px solid #2EAB3B;
    font-size: 15px;
    border-radius: 5px;
    margin-left: 10px;
    padding: 6px 15px;
    display: inline-block;
    margin-bottom: 5px;
    cursor: pointer;
  }
.caseListSure{
  color: #fff;
  background: #2EAB3B;
}
.creatCase{

}
.caseListTable{
  padding-top: 10px;
  margin-top: 20px;
  padding-bottom: 20px;
}
.caseListTable .tableTitle{
  margin-bottom: 10px;
}
.caseListTable .tableTitle span{
  padding-right: 5px;
}
  .caseListReset{
    color: #2EAB3B;
    background: #fff;
  }
  .table th{
    color: #2EAB3B;
    background: #ECF8F5;
    border: none;
  }
  .listAssign, .listView{
    color: #0D70D8;
    padding:6px;
    cursor: pointer;
  }

</style>
