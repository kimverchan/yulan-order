<template>
  <el-card class="centerCard">
    <el-dialog title="窗帘详情" :visible.sync="detailVisible" width="95%" top="5vh">
      <keep-alive>
        <detailCurtainTable v-if="detailVisible" :tableStatus="check_CURTAIN_STATUS_ID != 1 ? 3 : 2"
          :headerData="headerData" :curtainData="curtainData" :suggestion="ljsuggestion" @visible="closeTheDialog"
          @deleteArr="getDeleteArr" @finalData="getFinalData">
        </detailCurtainTable>
      </keep-alive>
    </el-dialog>
    <el-dialog title="查看购买凭证" width="700px" :visible.sync="buyUserPictureVisible">
      <div style="display: inline-block;margin:5px;cursor:pointer;" v-for="(file,index) in fileList" :key="index">
        <el-image style="width: 200px; height: 200px" :src="file" fit="fill" @click="handleImgClick(index)"
          :preview-src-list="fileList2"></el-image>
      </div>
    </el-dialog>

    <div slot="header">
      <span class="headSpan">订单详情</span>
      <el-button @click="exportProductExcel" v-if="check_CURTAIN_STATUS_ID == -1 && showExportProduct" size="mini"
        plain>
        导出生产模板
      </el-button>
      <el-button @click="backTowhere()" style="float:right;" size="small" type="success" plain v-if="button_1">返回
      </el-button>
    </div>
    <el-card class="tableCard" shadow="hover" body-style="padding:1px">
      <div slot="header">
        <span class="zoomLeft">
          订单号：
          <span class="zoomRight">{{ ruleForm.ORDER_NO }}</span>
        </span>
        <span class="zoomLeft">
          经办人：
          <span class="zoomRight">{{ ruleForm.LINKPERSON }}({{ ruleForm.TELEPHONE }})</span>
        </span>
        <span class="zoomLeft">
          收货人：
          <span class="zoomRight">{{ ruleForm.WL_CONTACTS }}({{ ruleForm.WL_TEL }})</span>
        </span>
        <span class="zoomLeft">
          收货地址：
          <span class="zoomRight">{{ ruleForm.ALL_ADDRESS }}</span>
        </span>

        <br />
        <span class="zoomLeft">
          购买人：
          <span class="zoomRight">{{ ruleForm.BUYUSER }}({{ ruleForm.BUYUSERPHONE }})</span>
        </span>
        <span class="zoomLeft">
          购买人地址：
          <span class="zoomRight">{{ ruleForm.BUYUSER_ADDRESS }}</span>
        </span>
        <span class="zoomLeft" v-if="ruleForm.BUYUSER_PICTURE">
          <el-link type="primary" @click="buyUserPictureVisible=true">查看购买凭证</el-link>
        </span>
        <span class="zoomLeft" v-if="ruleForm.PACKING_NOTE">
          分包备注：
          <span class="zoomRight">{{ ruleForm.PACKING_NOTE }}</span>
        </span>
        <br />
        <span class="zoomLeft">
          订单备注：
          <span class="zoomRight">{{ ruleForm.NOTES }}</span>
        </span>
        <span class="zoomLeft">
          玉兰处理说明：
          <span class="zoomRight">{{ ruleForm.YULAN_NOTES }}</span>
        </span>
      </div>
      <el-table border :show-summary="ruleForm.ORDERBODY.length > 1" :summary-method="getSummaries"
        :data="ruleForm.ORDERBODY" style="width: 100%" :row-class-name="tableRowClassName">
        <el-table-column align="center" prop="LINE_NO" label="序号" width="50"></el-table-column>
        <el-table-column align="center" prop="ITEM_NO" label="型号" width="140"></el-table-column>
        <el-table-column align="center" label="经销单价" width="80">
          <template slot-scope="scope1">
            <span v-if="isManager === '0' && check_CURTAIN_STATUS_ID > -1">***</span>
            <span v-else>{{ scope1.row.UNIT_PRICE | priceFilter }}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="QTY_REQUIRED" label="数量" width="60"></el-table-column>
        <el-table-column prop="PROMOTION" align="center" label="活动" width="110" show-overflow-tooltip></el-table-column>
        <el-table-column prop="PART_SEND_ID" align="center" :formatter="formatRole" label="发货说明" width="90">
        </el-table-column>
        <el-table-column prop="PROMOTION_COST" align="center" label="折后金额" width="90">
          <template slot-scope="scope1">
            <span v-if="isManager === '0' && check_CURTAIN_STATUS_ID > -1">***</span>
            <span v-else>{{ scope1.row.PROMOTION_COST }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="BACK_Y" align="center" label="年返利使用金额" width="90">
          <template slot-scope="scope1">
            <span v-if="isManager === '0' && check_CURTAIN_STATUS_ID >-1">***</span>
            <span v-else>{{ scope1.row.BACK_Y }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="BACK_M" align="center" label="月返利使用金额" width="90">
          <template slot-scope="scope1">
            <span v-if="isManager === '0' && check_CURTAIN_STATUS_ID > -1">***</span>
            <span v-else>{{ scope1.row.BACK_M }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="FINAL_COST" align="center" label="应付金额" width="90">
          <template slot-scope="scope1">
            <span v-if="isManager === '0' && check_CURTAIN_STATUS_ID != -1">***</span>
            <span v-else>{{ scope1.row.FINAL_COST }}</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="NOTES" label="备注"></el-table-column>
        <el-table-column v-if="isX" prop="LJ_SUGGESTION" align="center" label="兰居备注"></el-table-column>
        <el-table-column v-if="isX" align="center" label="窗帘详情" width="105">
          <template slot-scope="scope">
            <el-button @click="openDialog(scope.row, scope.$index)" type="primary" size="mini">查看详情</el-button>
          </template>
        </el-table-column>
        <el-table-column v-if="check_CURTAIN_STATUS_ID == 1 && isX" align="center" prop="checkStatus" label="是否修改"
          width="80"></el-table-column>
      </el-table>

      <div style="float:right;margin-top:20px;margin-right:10px;height:80px;">
        <el-button v-if="check_CURTAIN_STATUS_ID == 2" @click="_defeat()" size="medium" type="warning">退回兰居修改
        </el-button>
        <el-button v-if="check_CURTAIN_STATUS_ID == 2" @click="_pass()" size="medium" type="success">确认兰居修改</el-button>
        <el-button :disabled="exButton" v-if="check_CURTAIN_STATUS_ID == 1" @click="LjExamine()" size="medium"
          type="success">确认修改</el-button>
        <el-button v-if="
            (check_CURTAIN_STATUS_ID == 0 || check_CURTAIN_STATUS_ID == 4) &&
              check_STATUS_ID == 0
          " @click="summitCurtain" size="medium" type="primary">提交订单</el-button>
        <el-button v-if="check_STATUS_ID == 5 || check_STATUS_ID == 6" @click="refreshPay()" size="medium" type="danger"
          plain>提交订单</el-button>
      </div>
      <div style="padding:10px;">
        <span class="timeLeft">
          创建：
          <span class="timeRight">{{ ruleForm.DATE_CRE | datatrans}}</span>
        </span>
        <span v-if="ruleForm.WEB_TJ_TIME && ruleForm.WEB_TJ_TIME != '9999/12/31 00:00:00'" class="timeLeft">
          提交：
          <span class="timeRight">{{ ruleForm.WEB_TJ_TIME | datatrans}}</span>
        </span>
        <span class="timeLeft">
          更新：
          <span class="timeRight">{{ ruleForm.DATE_UPDATE | datatrans}}</span>
        </span>
        <br />
        <span v-if="ruleForm.DATE_ACCEPT && ruleForm.DATE_ACCEPT != '9999/12/31 00:00:00'" class="timeLeft">
          接收：
          <span class="timeRight">{{ ruleForm.DATE_ACCEPT | datatrans}}</span>
        </span>
        <span v-if="ruleForm.DATE_DEAL && ruleForm.DATE_DEAL != '9999/12/31 00:00:00'" class="timeLeft">
          处理：
          <span class="timeRight">{{ ruleForm.DATE_DEAL | datatrans }}</span>
        </span>
        <span v-if="ruleForm.USER_NO" class="timeLeft">
          处理人：
          <span class="timeRight">{{ ruleForm.USER_NO }}</span>
        </span>
      </div>
      <el-divider></el-divider>
      <div style="margin-top:0px;">
        <span style="margin-left:10px;color:red;">法律效力：本订单是双方合作协议不可分割的一部分，是乙方向甲方订货的凭证，具法力效力。</span>
      </div>
      <div v-if="operationRecords.length > 0" style="width:800px;margin-bottom:20px;">
        <h1 style="margin-left:10px;">处理记录：</h1>
        <el-steps direction="vertical" :active="operationRecords.length" style="margin-top:10px;margin-left:20px;">
          <el-step v-for="item in operationRecords" :key="item.value" style="margin-top:1px;">
            <template slot="title">
              <div v-html="item.OPERATION_NOTE"></div>
            </template>
          </el-step>
        </el-steps>
      </div>
    </el-card>
  </el-card>
</template>

<script>
import Axios from "axios";
import {
  getOrderlist,
  passExamine,
  orderDetail,
  defeatChange,
  queryCash,
  payAgain,
} from "@/api/orderList";
import {
  updateCurtainOrder,
  InsertOperationRecord,
  getOperationRecord,
  getOrderDetails,
  GetCtmOrder,
  GetPromotionByType,
  GetPromotionByTypeAndId,
  GetOrderUseRebate,
  ljExportProductExcel,
} from "@/api/orderListASP";
import { mapMutations, mapActions } from "vuex";
import { mapState } from "vuex";
import Cookies from "js-cookie";
import DetailCurtainTable from "../detail/detailCurtainTable";
import { async } from "q";
import { downLoadFile } from "@/common/js/downLoadFile";

export default {
  name: "examineDatail",
  props: ["isShowButton"],
  data() {
    return {
      button_1: true,
      deleteIds: [],
      defeatButton: false,
      exButton: true,
      ljsuggestion: "", //兰居审核意见
      cyLineNo: 0,
      allCurtains: [],
      headerData: {},
      isX: false,
      curtainData: "",
      Initial_balance: 0,
      renderArray: [],
      tableIndex: "",
      isManager: Cookies.get("isManager"),
      check_CURTAIN_STATUS_ID: "",
      check_STATUS_ID: "",
      orderNum: "",
      detailVisible: false,
      buyUserPictureVisible: false,
      fileList: [],
      fileList2: [],
      showViewer: false,
      operationRecords: [],
      ruleForm: {
        ORDER_NO: "",
        LINKPERSON: "",
        WL_CONTACTS: "",
        WL_TEL: "",
        POST_ADDRESS: "",
        NOTES: "",
        ORDERBODY: [],
      },
      showExportProduct: false,
    };
  },
  components: {
    DetailCurtainTable,
  },
  filters: {
    priceFilter(value) {
      //四舍五入过滤大法
      let realVal = parseFloat(value).toFixed(2);
      //防止出现-0.00；
      if (realVal <= 0) {
        realVal = 0.0;
      }

      return realVal;
    },
    datatrans(value) {
      if (!value || value == "9999/12/31 00:00:00") {
        return "";
      }
      //时间戳转化大法
      let date = new Date(value);
      let y = date.getFullYear();
      let MM = date.getMonth() + 1;
      MM = MM < 10 ? "0" + MM : MM;
      let d = date.getDate();
      d = d < 10 ? "0" + d : d;
      let h = date.getHours();
      h = h < 10 ? "0" + h : h;
      let m = date.getMinutes();
      m = m < 10 ? "0" + m : m;
      let s = date.getSeconds();
      s = s < 10 ? "0" + s : s;
      return y + "-" + MM + "-" + d + " " + h + ":" + m + ":" + s;
    },
  },
  created: function () {
    this.orderNum = Cookies.get("ORDER_NO");
    this.isX = this.orderNum.slice(0, 1) == "X";
    this.check_CURTAIN_STATUS_ID = Cookies.get("CURTAIN_STATUS_ID");
    this.check_STATUS_ID = Cookies.get("status_ID");
    this.getDetail();
    if (this.isShowButton != undefined) {
      this.button_1 = this.isShowButton;
    }
  },
  activated() {
    if (this.isShowButton != undefined) {
      this.orderNum = Cookies.get("ORDER_NO");
      this.isX = this.orderNum.slice(0, 1) == "X";
      this.check_CURTAIN_STATUS_ID = Cookies.get("CURTAIN_STATUS_ID");
      this.check_STATUS_ID = Cookies.get("status_ID");
      this.getDetail();
    }
  },
  methods: {
    formatRole: function (row, column) {
      if (row.PART_SEND_ID == 0) {
        return "等生产";
      } else if (row.PART_SEND_ID == 1) {
        return "分批发货";
      } else return "--";
    },
    closeTheDialog(msg) {
      this.detailVisible = msg;
    },
    getDeleteArr(msg) {
      for (var i = 0; i < msg.length; i++) {
        this.deleteIds.push(msg[i]);
      }
    },
    //获取修改后的表格数据
    getFinalData(msg) {
      if (msg != null) {
        this.saveChange();
      }
      let innerLine = msg[0].lineNo;
      this.allCurtains.push(msg);
      //换掉的item赋值
      for (let i = 0; i < msg.length; i++) {
        msg[i].itemId = msg[i].item.itemNo;
      }
      for (let a = 0; a < this.allCurtains.length - 1; a++) {
        if (this.allCurtains[a][0].lineNo == innerLine) {
          this.allCurtains.splice(a, 1);
        }
      }
    },
    summitCurtain() {
      var item = this.ruleForm;
      let orderBody = item.ORDERBODY;
      let transCookies = [];
      for (let i = 0; i < orderBody.length; i++) {
        transCookies[i] = new Object();
        transCookies[i].width = orderBody[i].CURTAIN_WIDTH;
        transCookies[i].height = orderBody[i].CURTAIN_HEIGHT;
        transCookies[i].orderNumber = item.ORDER_NO;
        transCookies[i].lineNo = orderBody[i].LINE_NO;
        transCookies[i].activityId = orderBody[i].curtains[0].activityId;
        transCookies[i].quantity = orderBody[i].QTY_REQUIRED;
        var price = 0;
        for (let j = 0; j < orderBody[i].curtains.length; j++) {
          price += orderBody[i].curtains[j].price.mul(
            orderBody[i].curtains[j].dosage
          );
        }
        transCookies[i].price = price;
        transCookies[i].splitShipment = orderBody[i].PART_SEND_ID;
        transCookies[i].activityName = orderBody[i].PROMOTION;
        transCookies[i].unit = orderBody[i].UNIT;
        transCookies[i].item = orderBody[i].item;
      }
      sessionStorage.setItem("shopping", JSON.stringify(transCookies));
      sessionStorage.setItem("shoppingHead", JSON.stringify(item));
      Cookies.set("cur_status", 3);
      this.addTab("order/checkOrder");
      this.closeTab("order/checkExamine");
    },
    //客户修改
    LjExamine() {
      let url = "/order/updateCurtainOrder.do";
      let data = {
        cid: Cookies.get("cid"),
        orderNo: this.orderNum,
        curtainStatusId: "0",
        allCurtains: this.allCurtains,
        deleteIds: this.deleteIds,
      };
      //defeatChange(url,data).then(res =>{
      updateCurtainOrder(data)
        .then((res) => {
          if (res.code == 0) {
            this.$alert("操作成功,请提交结算再次审核", "提示", {
              confirmButtonText: "确定",
              type: "success",
            });
            this.check_CURTAIN_STATUS_ID = "0";
            this.getDetail();
          } else {
            this.$alert("操作失败，请稍后重试", "提示", {
              confirmButtonText: "确定",
              type: "warning",
            });
          }
        })
        .catch((res) => {
          this.$alert("操作失败:" + res.msg, "提示", {
            confirmButtonText: "确定",
            type: "warning",
          });
        });
    },
    //确认兰居修改，通过订单审核变为可提交状态
    _pass() {
      var url = "/order/updateCurOrderStatus.do";
      var data = {
        orderNo: this.orderNum,
        curtainStatusId: "4",
      };
      this.$confirm("确认同意兰居修改？", "提示", {
        confirmButtonText: "是",
        cancelButtonText: "否",
        type: "info",
      }).then(() => {
        passExamine(url, data)
          .then((res) => {
            if (res.code == 0) {
              var recordData = {
                ORDER_NO: this.orderNum,
                OPERATION_PERSON: Cookies.get("cid"),
                OPERATION_NAME: "确认兰居修改",
              };
              InsertOperationRecord(recordData); //插入操作记录
              this.$alert("操作成功,该订单已经确认,可再次提交", "提示", {
                confirmButtonText: "确定",
                type: "success",
              }).then(() => {
                this.check_CURTAIN_STATUS_ID = "4";
              });
            } else {
              this.$alert("操作失败，请稍后重试", "提示", {
                confirmButtonText: "确定",
                type: "warning",
              });
            }
          })
          .catch((res) => {
            this.$alert("操作失败，请稍后重试", "提示", {
              confirmButtonText: "确定",
              type: "warning",
            });
          });
      });
    },
    //退回兰居修改
    _defeat() {
      var url = "/order/updateCurOrderStatus.do";
      var data = {
        orderNo: this.orderNum,
        curtainStatusId: "3",
      };
      this.$confirm("确定将订单退回兰居重新修改？", "提示", {
        confirmButtonText: "是",
        cancelButtonText: "否",
        type: "info",
      }).then(() => {
        passExamine(url, data)
          .then((res) => {
            if (res.code == 0) {
              var recordData = {
                ORDER_NO: this.orderNum,
                OPERATION_PERSON: Cookies.get("cid"),
                OPERATION_NAME: "退回兰居修改",
              };
              InsertOperationRecord(recordData); //插入操作记录
              this.$alert("操作成功,该订单已退回兰居修改", "提示", {
                confirmButtonText: "确定",
                type: "success",
              }).then(() => {
                this.closeToTab({
                  oldUrl: "order/checkExamine",
                  newUrl: "order/myOrder",
                });
              });
            } else {
              this.$alert("操作失败，请稍后重试", "提示", {
                confirmButtonText: "确定",
                type: "warning",
              });
            }
          })
          .catch((res) => {
            this.$alert("操作失败，请稍后重试", "提示", {
              confirmButtonText: "确定",
              type: "warning",
            });
          });
      });
    },
    openDialog(tab, index) {
      this.ljsuggestion = tab.LJ_SUGGESTION;
      this.tableIndex = index;
      this.cyLineNo = index + 1;
      this.detailVisible = true;
      for (let i = 0; i < tab.curtains.length; i++) {
        tab.curtains[i].choose = true;
        tab.curtains[i].productType = tab.curtains[i].item.productType;
        tab.curtains[i].itemType = tab.curtains[i].curtainPartName;
      }
      this.curtainData = tab.curtains;
      for (let i = 0; i < this.allCurtains.length; i++) {
        if (this.cyLineNo == this.allCurtains[i][0].lineNo) {
          this.curtainData = this.allCurtains[i];
        }
      }
      this.headerData.quantity = tab.QTY_REQUIRED;
      this.headerData.highJia = tab.CURTAIN_HEIGHT2;
      this.headerData.productGroupType = ""; //
      this.headerData.price = tab.all_cost;
      if (tab.CURTAIN_WBH_SIZE == -1) {
        this.headerData.outsourcingBoxExist = 0;
      } else {
        this.headerData.outsourcingBoxExist = 1;
      }
      this.headerData.outsourcingBoxWidth = tab.CURTAIN_WBH_SIZE;
      this.headerData.modelNumber = tab.ITEM_NO;
      this.headerData.location = tab.CURTAIN_ROOM_NAME
        ? tab.CURTAIN_ROOM_NAME
        : "无";
      this.headerData.height = tab.CURTAIN_HEIGHT;
      this.headerData.width = tab.CURTAIN_WIDTH;
      this.headerData.drape = tab.CURTAIN_SIZE_TIMES;
      this.headerData.checked = "";
      this.headerData.cartItemId = "";
      this.headerData.activityGroupType = "";
      this.headerData.activity = tab.PROMOTION_TYPE;
      this.headerData.activityId = tab.curtains[0].activityId;
      this.headerData.activityName = tab.PROMOTION;
    },
    //保存修改
    saveChange() {
      this.renderArray = this.ruleForm.ORDERBODY[this.tableIndex];
      this.renderArray.checkStatus = "已修改";
      this.Render();
      this.detailVisible = false;
      this.exButton = false;
      this.defeatButton = true;
    },
    //渲染
    Render() {
      this.$set(this.ruleForm.ORDERBODY, this.tableIndex, this.renderArray);
      this.passORback = true;
    },
    getDetail() {
      getOrderDetails({ orderNo: this.orderNum }).then((res) => {
        this.ruleForm = res.data[0];
        if (
          this.ruleForm.CUSTOMER_CODE == "C01613" ||
          this.ruleForm.CUSTOMER_CODE == "C01613A"
        ) {
          this.showExportProduct = true;
        }
        if (this.ruleForm.BUYUSER_PICTURE) {
          var list = this.ruleForm.BUYUSER_PICTURE.split(";");
          for (var i = 0; i < list.length - 1; i++) {
            var index = list[i].lastIndexOf("/");
            if (index == -1) index = list[i].lastIndexOf("\\");
            var fileName = list[i].substr(index + 1);
            this.fileList.push(this.Global.baseUrl + list[i]);
          }
        }
        for (let i = 0; i < this.ruleForm.ORDERBODY.length; i++) {
          this.ruleForm.ORDERBODY[i].checkStatus = "未修改";
        }
        var recordData = {
          orderNo: this.orderNum,
        };
        getOperationRecord(recordData).then((res2) => {
          this.operationRecords = res2.data;
        });
      });
    },
    //返回指定
    backTowhere() {
      let customerType = Cookies.get("customerType");
      if (customerType == 110) {
        this.addTab("order/examine");
      } else {
        this.addTab("order/myOrder");
      }
    },
    refreshPay() {
      var url = "/order/getResidemoney.do";
      var data = {
        cid: Cookies.get("cid"),
        companyId: Cookies.get("companyId"),
      };
      //每次重新提交的时候判断一下余额
      queryCash(url, data).then(async (res) => {
        this.Initial_balance = res.data;
        var url2 = "/order/putAgainOrder.do";
        var data2 = {
          cid: Cookies.get("cid"),
          orderNo: this.orderNum,
        };
        if (
          this.ruleForm.ALL_SPEND > this.Initial_balance &&
          this.check_STATUS_ID == 5
        ) {
          //欠款可提交的话可以跳过判断
          this.$alert(
            "余额不足，当前订单还需充值" +
              (this.ruleForm.ALL_SPEND - this.Initial_balance) +
              "元才能提交",
            "提示",
            {
              confirmButtonText: "确定",
              type: "warning",
            }
          );
        } else {
          if (this.ruleForm.STATUS_ID == 5 || this.ruleForm.STATUS_ID == 6) {
            //欠款可提交判断活动和优惠券是否过期
            for (var i = 0; i < this.ruleForm.ORDERBODY.length; i++) {
              if (
                this.ruleForm.ORDERBODY[i].PROMOTION_TYPE &&
                this.ruleForm.ORDERBODY[i].PROMOTION_TYPE != " "
              ) {
                var res = await GetPromotionByTypeAndId({
                  proType: this.ruleForm.ORDERBODY[i].PROMOTION_TYPE,
                  pId: this.ruleForm.ORDERBODY[i].P_ID,
                  cid: Cookies.get("cid"),
                });
                if (!res.data) {
                  this.$alert(
                    `活动‘${this.ruleForm.ORDERBODY[i].PROMOTION}’不存在`,
                    "提示",
                    {
                      confirmButtonText: "确定",
                      type: "success",
                    }
                  );
                  return;
                }
                var dateEnd = new Date(res.data.DATE_END);
                dateEnd = dateEnd.setDate(dateEnd.getDate() + 1);
                if (new Date(dateEnd) < new Date() || res.data.USE_ID == "0") {
                  this.$alert(
                    `活动‘${this.ruleForm.ORDERBODY[i].PROMOTION}’已过期，请删除订单后重新下单`,
                    "提示",
                    {
                      confirmButtonText: "确定",
                      type: "success",
                    }
                  );
                  return;
                }
              }
              if (
                this.ruleForm.ORDERBODY[i].BACK_Y > 0 ||
                this.ruleForm.ORDERBODY[i].BACK_M > 0
              ) {
                var res = await GetOrderUseRebate({
                  orderNo: this.ruleForm.ORDERBODY[i].ORDER_NO,
                  lineNo: this.ruleForm.ORDERBODY[i].LINE_NO,
                });
                if (res.data.length == 0) {
                  this.$alert(`优惠券不存在`, "提示", {
                    confirmButtonText: "确定",
                    type: "success",
                  });
                  return;
                }
                for (var j = 0; j < res.data.length; j++) {
                  if (new Date(res.data[j].DATE_END) < new Date()) {
                    this.$alert(`优惠券已过期，请删除订单后重新下单`, "提示", {
                      confirmButtonText: "确定",
                      type: "success",
                    });
                    return;
                  }
                }
              }
            }
          }
          payAgain(url2, data2)
            .then((res) => {
              var recordData = {
                ORDER_NO: this.orderNum,
                OPERATION_PERSON: Cookies.get("cid"),
                OPERATION_NAME: "重新提交",
              };
              InsertOperationRecord(recordData); //插入操作记录
              this.$alert("提交成功", "提示", {
                confirmButtonText: "确定",
                type: "success",
              });
              this.$root.$emit("refreshMoneyEvent"); //触发主页面刷新余额
              this.addTab("order/myOrder");
            })
            .catch((res) => {
              this.$alert("操作失败，请稍后重试", "提示", {
                confirmButtonText: "确定",
                type: "warning",
              });
            });
        }
      });
    },
    handleImgClick(index) {
      let tempImgList = [...this.fileList];
      let temp = [];
      for (let i = 0; i < index; i++) {
        temp.push(tempImgList.shift());
      }
      this.fileList2 = tempImgList.concat(temp);
      this.showViewer = true;
    },
    exportProductExcel() {
      this.$confirm(
        "此功能只适用于导出测试账号订单，导出模板后订单状态将变成已完成状态，请慎重操作，是否确认导出？",
        "提示",
        {
          confirmButtonText: "确定",
          type: "warning",
        }
      )
        .then(() => {
          ljExportProductExcel({
            cid: Cookies.get("cid"),
            orderNo: this.orderNum,
          }).then((res) => {
            if (res.msg) {
              downLoadFile(
                this.Global.baseUrl + `DownLoadAPI/DownloadFile?path=${res.msg}`
              );
              this.closeToTab({
                oldUrl: "order/checkExamine",
                newUrl: "order/examine",
              });
            }
          });
        })
        .catch(() => {
          return;
        });
    },
    //合计行显示
    getSummaries({ columns, data }) {
      var sums = [];
      columns.forEach((column, index) => {
        if (index == 0) {
          sums[index] = "总计";
          return;
        } else if (index == 6 || index == 7 || index == 8 || index == 9) {
          var values = data.map((item) => Number(item[column.property]));
          var cancelIndex = data.map((item) => Number(item["STATUS_ID"])); //取得状态判断是否是作废的
          if (!values.every((value) => isNaN(value))) {
            sums[index] = values.reduce((prev, curr, index) => {
              const value = Number(curr);
              if (!isNaN(value) && cancelIndex[index] != "3") {
                //作废的不统计
                return prev + curr;
              } else {
                return prev;
              }
            }, 0);
            sums[index] = sums[index].toFixed(2);
            if (this.isManager == "0" && this.check_CURTAIN_STATUS_ID != -1)
              sums[index] = "***";
          } else {
            sums[index] = "";
          }
        } else {
          sums[index] = "";
        }
      });
      return sums;
    },
    //隔行变色
    tableRowClassName({ row, rowIndex }) {
      if (row.STATUS_ID == "3") {
        return "fuck-row";
      }
      if (rowIndex % 2 == 0) {
        return "success-row";
      }
      return "";
    },
    ...mapMutations("navTabs", ["addTab"]),
    ...mapActions("navTabs", ["closeTab", "closeToTab"]),
  },
};
</script>

<style scoped>
.centerCard {
  margin: 0 auto;
  position: relative;
}
.childCard {
  width: 90%;
  height: 200px;
  float: left;
}
.tableCard {
  float: left;
  width: 100%;
}
.headSpan {
  font-weight: bold;
  font-size: 18px;
  color: black;
}
.zoomRight {
  font-size: 15px;
  line-height: 30px;
  display: inline-block;
  margin-right: 30px;
  font-weight: bold;
}
.zoomLeft {
  font-size: 15px;
  line-height: 30px;
  display: inline-block;
  margin-right: 10px;
}
.timeLeft {
  font-size: 14px;
  line-height: 30px;
  display: inline-block;
}
.timeRight {
  font-size: 14px;
  line-height: 30px;
  display: inline-block;
  margin-right: 20px;
  font-weight: bold;
}
</style>
<style>
.el-table .success-row {
  background: #f0f9eb;
}
.el-table .fuck-row {
  background: #f0f9eb;
  color: tomato;
  text-decoration: line-through;
}
.centerCard .el-dialog__body {
  padding: 10px;
}
</style>