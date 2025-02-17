<template>
  <div class="dataset-on-time">
    <el-row class="top-operate">
      <el-col :span="10">
        <deBtn
          type="primary"
          icon="el-icon-circle-plus-outline"
          @click="() => selectDataset()"
          >{{ $t("dataset.add_task") }}</deBtn
        >
        <deBtn
          :disabled="!multipleSelection.length"
          secondary
          @click="confirmDelete"
          >{{ $t("organization.delete") }}</deBtn
        >
      </el-col>
      <el-col :span="14" class="right-user">
        <el-input
          :placeholder="$t('通过任务名称搜索')"
          prefix-icon="el-icon-search"
          class="name-email-search"
          size="small"
          clearable
          ref="search"
          v-model="nikeName"
          @blur="initSearch"
          @clear="initSearch"
        >
        </el-input>
        <deBtn
          :secondary="!filterTexts.length"
          :plain="!!filterTexts.length"
          icon="iconfont icon-icon-filter"
          @click="filterShow"
          >{{ $t("user.filter")
          }}<template v-if="filterTexts.length">
            ({{ filterTexts.length }})
          </template>
        </deBtn>
        <el-dropdown trigger="click" :hide-on-click="false">
          <deBtn secondary icon="el-icon-setting">{{ $t("user.list") }}</deBtn>
          <el-dropdown-menu class="list-colums-slect" slot="dropdown">
            <p class="title">{{ $t("user.list_info") }}</p>
            <el-checkbox
              :indeterminate="isIndeterminate"
              v-model="checkAll"
              @change="handleCheckAllChange"
              >{{ $t("dataset.check_all") }}</el-checkbox
            >
            <el-checkbox-group
              v-model="checkedColumnNames"
              @change="handleCheckedColumnNamesChange"
            >
              <el-checkbox
                v-for="column in columnNames"
                :label="column.props"
                :key="column.props"
                >{{ $t(column.label) }}</el-checkbox
              >
            </el-checkbox-group>
          </el-dropdown-menu>
        </el-dropdown>
      </el-col>
    </el-row>
    <div class="filter-texts" v-if="filterTexts.length">
      <span class="sum">{{ paginationConfig.total }}</span>
      <span class="title">{{ $t("user.result_one") }}</span>
      <el-divider direction="vertical"></el-divider>
      <i
        @click="scrollPre"
        v-if="showScroll"
        class="el-icon-arrow-left arrow-filter"
      ></i>
      <div class="filter-texts-container">
        <p class="text" v-for="(ele, index) in filterTexts" :key="ele">
          {{ ele }} <i @click="clearOneFilter(index)" class="el-icon-close"></i>
        </p>
      </div>
      <i
        @click="scrollNext"
        v-if="showScroll"
        class="el-icon-arrow-right arrow-filter"
      ></i>
      <el-button
        type="text"
        class="clear-btn"
        icon="el-icon-delete"
        @click="clearFilter"
        >{{ $t("user.clear_filter") }}</el-button
      >
    </div>
    <div class="table-container"
      id="resize-for-filter"
      :class="[filterTexts.length ? 'table-container-filter' : '']">
      <grid-table
        v-loading="$store.getters.loadingMap[$store.getters.currentPath]"
        :tableData="data"
        :columns="checkedColumnNames"
        :multipleSelection="multipleSelection"
        :pagination="paginationConfig"
        @selection-change="handleSelectionChange"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      >
        <el-table-column type="selection" width="55" />
        <el-table-column
          width="178"
          prop="name"
          :label="$t('dataset.task_name')"
        >
          <template slot-scope="scope">
            <span>
              <el-link
                style="font-size: 12px"
                @click="jumpTaskRecord(scope.row)"
                >{{ scope.row.name }}</el-link
              >
            </span>
          </template>
        </el-table-column>
        <el-table-column
          width="178"
          prop="datasetName"
          :label="$t('dataset.task.dataset')"
        />
        <el-table-column
          width="100"
          prop="rate"
          :label="$t('dataset.execute_rate')"
        >
          <template slot-scope="scope">
            <span v-if="scope.row.rate === 'SIMPLE'">{{
              $t("dataset.execute_once")
            }}</span>
            <span v-if="scope.row.rate === 'CRON'">{{
              $t("dataset.cron_config")
            }}</span>
            <span v-if="scope.row.rate === 'SIMPLE_CRON'">{{
              $t("dataset.simple_cron")
            }}</span>
          </template>
        </el-table-column>

        <el-table-column
          prop="lastExecTime"
          width="178"
          :label="$t('dataset.task.last_exec_time')"
        >
          <template slot-scope="scope">
            <span>
              {{ scope.row.lastExecTime | timestampFormatDate }}
            </span>
          </template>
        </el-table-column>

        <el-table-column
          prop="lastExecStatus"
          width="140"
          :label="$t('dataset.task.last_exec_status')"
        >
          <template slot-scope="scope">
            <span :class="[`de-${scope.row.lastExecStatus}`, 'de-status']"
              >{{
                $t(`dataset.${scope.row.lastExecStatus.toLocaleLowerCase()}`)
              }}
              <i
                v-if="scope.row.lastExecStatus === 'Error'"
                class="el-icon-s-order"
                @click="showErrorMassage(scope.row.msg)"
              ></i>
            </span>
          </template>
        </el-table-column>

        <el-table-column
          prop="nextExecTime"
          width="178"
          :label="$t('dataset.task.next_exec_time')"
        >
          <template slot-scope="scope">
            <span
              v-if="
                scope.row.nextExecTime &&
                scope.row.nextExecTime !== -1 &&
                scope.row.rate !== 'SIMPLE' &&
                scope.row.status !== 'Pending'
              "
            >
              {{ scope.row.nextExecTime | timestampFormatDate }}
            </span>
            <span v-if="!scope.row.nextExecTime || scope.row.rate === 'SIMPLE'"
              >-</span
            >
          </template>
        </el-table-column>

        <el-table-column
          min-width="120"
          prop="status"
          :label="$t('dataset.task.task_status')"
        >
          <template slot-scope="scope">
            <span :class="[`de-${scope.row.status}`, 'de-status']"
              >{{ $t(`dataset.task.${scope.row.status.toLocaleLowerCase()}`) }}
            </span>
          </template>
        </el-table-column>
        <el-table-column
          slot="__operation"
          :label="$t('commons.operating')"
          fixed="right"
          width="84"
        >
          <template slot-scope="scope">
            <el-button
              @click="selectDataset(scope.row)"
              class="text-btn mar3 mar6"
              type="text"
              >{{
                $t(disableEdit(scope.row) ? "auth.view" : "commons.edit")
              }}</el-button
            >
            <el-dropdown
              size="medium"
              trigger="click"
              @command="(type) => handleCommand(type, scope.row)"
            >
              <i @click.stop class="el-icon-more"></i>
              <el-dropdown-menu class="de-card-dropdown" slot="dropdown">
                <template
                  v-if="!['Stopped', 'Exec'].includes(scope.row.status)"
                >
                  <el-dropdown-item
                    :disabled="disableExec(scope.row)"
                    command="exec"
                  >
                    {{ $t("运行一次") }}
                  </el-dropdown-item>
                  <el-dropdown-item
                    v-if="scope.row.status === 'Pending'"
                    command="contine"
                  >
                    {{ $t("继续") }}
                  </el-dropdown-item>
                  <el-dropdown-item
                    v-if="scope.row.status === 'Underway'"
                    command="pause"
                  >
                    {{ $t("dataset.task.pending") }}
                  </el-dropdown-item>
                </template>
                <el-dropdown-item
                  :disabled="disableDelete(scope.row)"
                  command="delete"
                >
                  {{ $t("commons.delete") }}
                </el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </template>
        </el-table-column>
      </grid-table>
    </div>


    <keep-alive>
      <filterUser ref="filterUser" @search="filterDraw"></filterUser>
    </keep-alive>

    <el-dialog
      v-dialogDrag
      :title="$t('dataset.error') + $t('dataset.detail')"
      :visible.sync="show_error_massage"
      width="600px"
      class="de-dialog-form"
    >
      <span class="err-msg">{{ error_massage }}</span>
      <span slot="footer" class="dialog-footer">
        <deBtn secondary @click="show_error_massage = false">{{
          $t("dataset.close")
        }}</deBtn>
      </span>
    </el-dialog>
  </div>
</template>

<script>
const columnOptions = [
  {
    label: "dataset.task_name",
    props: "name",
  },
  {
    label: "dataset.task.dataset",
    props: "datasetName",
  },
  {
    label: "dataset.execute_rate",
    props: "rate",
  },
  {
    label: "dataset.task.last_exec_time",
    props: "lastExecTime",
  },
  {
    label: "dataset.task.last_exec_status",
    props: "lastExecStatus",
  },
  {
    label: "dataset.task.next_exec_time",
    props: "nextExecTime",
  },
  {
    label: "dataset.task.task_status",
    props: "status",
  },
];
import {
  formatOrders,
} from "@/utils/index";
import { datasetTaskList, post } from "@/api/dataset/dataset";
import cron from "@/components/cron/cron";
import TableSelector from "@/views/chart/view/TableSelector";
import { hasDataPermission } from "@/utils/permission";
import GridTable from "@/components/gridTable/index.vue";
import filterUser from "./filterUser.vue";
import msgCfm from "@/components/msgCfm/index";
import _ from 'lodash';

export default {
  name: "DatasetTaskList",
  components: { GridTable, cron, filterUser, TableSelector },
  mixins: [msgCfm],
  data() {
    return {
      nikeName: "",
      showScroll: false,
      checkAll: true,
      multipleSelection: [],
      checkedColumnNames: columnOptions.map((ele) => ele.props),
      columnNames: columnOptions,
      isIndeterminate: false,
      filterTexts: [],
      paginationConfig: {
        currentPage: 1,
        pageSize: 10,
        total: 0,
      },
      cacheCondition: [],
      data: [],
      orderConditions: [],
      selectDatasetFlag: false,
      table: {},
      show_error_massage: false,
      error_massage: "",
      customType: ["db", "sql", "api"],
    };
  },
  watch: {
    filterTexts: {
      handler() {
        this.getScrollStatus();
      },
      deep: true,
    },
  },
  created() {
    this.initSearch();
    this.timer = setInterval(() => {
      this.search(false);
    }, 10000);
  },
  beforeDestroy() {
    clearInterval(this.timer);
  },
  mounted() {
    this.resizeObserver();
  },
  methods: {
    getScrollStatus() {
      this.$nextTick(() => {
        const dom = document.querySelector(".filter-texts-container");
        this.showScroll = dom && dom.scrollWidth > dom.offsetWidth;
      });
    },
    resizeObserver() {
      this.resizeForFilter = new ResizeObserver(entries => {
        if (!this.filterTexts.length) return;
        this.layoutResize();
      });
      this.resizeForFilter.observe(document.querySelector('#resize-for-filter'));
    },
    layoutResize: _.debounce(function () {
      this.getScrollStatus()
    }, 200),
    scrollPre() {
      const dom = document.querySelector('.filter-texts-container');
      dom.scrollLeft -= 10
      if (dom.scrollLeft <= 0) {
        dom.scrollLeft = 0
      }
    },
    scrollNext() {
      const dom = document.querySelector('.filter-texts-container');
      dom.scrollLeft += 10
      const width = dom.scrollWidth - dom.offsetWidth
      if (dom.scrollLeft > width) {
        dom.scrollLeft = width
      }
    },
    clearFilter() {
      this.$refs.filterUser.clearFilter();
    },
    clearOneFilter(index) {
      this.$refs.filterUser.clearOneFilter(index);
      this.$refs.filterUser.search();
    },
    filterDraw(condition, filterTexts = []) {
      this.cacheCondition = condition;
      this.filterTexts = filterTexts;
      this.initSearch();
    },
    filterShow() {
      this.$refs.filterUser.init();
    },
    handleCommand(key, row) {
      switch (key) {
        case "exec":
          this.execTask(row);
          return;
          break;
        case "delete":
          this.deleteTask(row);
          return;
          break;
        default:
          break;
      }
      this.changeTaskStatus(row);
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleCheckAllChange(val) {
      this.checkedColumnNames = val
        ? columnOptions.map((ele) => ele.props)
        : [];
      this.isIndeterminate = false;
    },
    handleCheckedColumnNamesChange(value) {
      let checkedCount = value.length;
      this.checkAll = checkedCount === this.columnNames.length;
      this.isIndeterminate =
        checkedCount > 0 && checkedCount < this.columnNames.length;
    },
    handleSizeChange(pageSize) {
      this.paginationConfig.currentPage = 1;
      this.paginationConfig.pageSize = pageSize;
      this.search();
    },
    handleCurrentChange(currentPage) {
      this.paginationConfig.currentPage = currentPage;
      this.search();
    },
    initSearch() {
      this.handleCurrentChange(1);
    },
    search(showLoading = true) {
      const param = {
        orders: formatOrders(this.orderConditions),
        conditions: [...this.cacheCondition],
      };
      if (this.nikeName) {
        param.conditions.push({
          field: `dataset_table_task.name`,
          operator: "like",
          value: this.nikeName,
        });
      }
      const { currentPage, pageSize } = this.paginationConfig;
      datasetTaskList(currentPage, pageSize, param, showLoading).then(
        (response) => {
          this.data = response.data.listObject;
          // this.data.forEach((item) => {
          //   this.taskStatus(item);
          // });
          this.paginationConfig.total = response.data.itemCount;
        }
      );
    },
    batchDelete() {
      post("/dataset/task/batchDelete", this.multipleSelection.map(ele => ele.id), false).then(() => {
        this.initSearch();
        this.openMessageSuccess('commons.delete_success');
      });
    },
    confirmDelete() {
      const options = {
          title: '确定删除该任务吗？',
          type: "primary",
          cb: this.batchDelete,
        };
        this.handlerConfirm(options);
    },
    taskStatus(item) {
      post("/dataset/task/lastExecStatus", item, false).then((response) => {
        if (!item.lastExecStatus) {
          item.lastExecStatus = response.data.lastExecStatus;
        }
        if (!item.lastExecTime) {
          item.lastExecTime = response.data.lastExecTime;
        }
        item.msg = response.data.msg;
      });
    },
    changeTaskStatus(task) {
      const { status } = task;
      if (!["Pending", "Underway"].includes(status)) {
        return;
      }
      const param = {
        ...task,
        status: status === "Underway" ? "Pending" : "Underway",
      };
      post("/dataset/task/updateStatus", param)
        .then((response) => {
          if (response.success) {
            task.status = param.status;
            this.$message({
              message: this.$t("dataset.task.change_success"),
              type: "success",
              showClose: true,
            });
          } else {
            this.initSearch(false);
          }
        })
        .catch(() => {
          this.initSearch(false);
        });
    },
    execTask(task) {
      this.$confirm(
        this.$t("dataset.task.confirm_exec"),
        this.$t("dataset.tips"),
        {
          confirmButtonText: this.$t("dataset.confirm"),
          cancelButtonText: this.$t("dataset.cancel"),
          type: "warning",
        }
      )
        .then(() => {
          post("/dataset/task/execTask", task).then((response) => {
            this.initSearch( true);
          });
        })
        .catch(() => {});
    },
    selectDataset(row) {
      if (row) {
        const { datasetName,  id } = row;
        this.$router.push({
          path: '/task-ds-form',
          query: {
            datasetName,
            id,
          }
        })
      } else {
        this.$router.push('/task-ds-form')
      }
    },
    disableEdit(task) {
      return (
        task.rate === "SIMPLE" ||
        task.status === "Stopped" ||
        !hasDataPermission("manage", task.privileges)
      );
    },
    disableExec(task) {
      return (
        task.status === "Stopped" ||
        task.status === "Pending" ||
        task.rate === "SIMPLE" ||
        !hasDataPermission("manage", task.privileges)
      );
    },
    disableDelete(task) {
      return false;
      // !hasDataPermission('manage',task.privileges)
    },
    deleteTask(task) {
      const options = {
          title: '确定删除该任务吗？',
          type: "primary",
          cb: () => {
            post("/dataset/task/delete/" + task.id, null).then((response) => {
            this.openMessageSuccess('commons.delete_success');
            this.initSearch();
          });
          },
        };
        this.handlerConfirm(options);
    },
    showErrorMassage(massage) {
      this.show_error_massage = true;
      this.error_massage = massage;
    },
    jumpTaskRecord(item) {
      this.$emit("jumpTaskRecord", item);
    },
  },
};
</script>

<style scoped>
.codemirror {
  height: 100px;
  overflow-y: auto;
}
.codemirror ::v-deep .CodeMirror-scroll {
  height: 100px;
  overflow-y: auto;
}

.err-msg {
  font-size: 12px;
  word-break: normal;
  width: auto;
  display: block;
  white-space: pre-wrap;
  word-wrap: break-word;
  overflow: hidden;
}

span {
  font-size: 12px;
}
</style>
<style lang="scss" scoped>
.dataset-on-time {
  margin: 0;
  width: 100%;
  overflow: auto;
  background-color: var(--ContentBG, #fff);
  padding: 24px;
  height: 100%;

  ::v-deep.el-link--inner {
    color: var(--primary, #3370ff) !important;
  }

  .el-icon-more {
    width: 24px;
    height: 24px;
    line-height: 24px;
    text-align: center;
    font-size: 12px;
    color: var(--primary, #3370ff);
  }

  .el-icon-more:hover {
    background: var(--deWhiteHover, #3370ff);
    border-radius: 4px;
  }

  .el-icon-more:active {
    background: var(--deWhiteActive, #3370ff);
    border-radius: 4px;
  }
}
.table-container {
  height: calc(100% - 50px);

  .text-btn {
    font-family: PingFang SC;
    font-size: 14px;
    font-weight: 400;
    line-height: 22px;
    letter-spacing: 0px;
    text-align: center;
    margin-left: 2px;
    border: none;
    padding: 2px 4px;
  }

  .text-btn:hover {
    background: rgba(51, 112, 255, 0.1);
  }

  .mar6 {
    margin-right: 6px;
  }

  .mar3 {
     margin-left: -3px;
  }
}

.table-container-filter {
  height: calc(100% - 110px);
}
.filter-texts {
  display: flex;
  align-items: center;
  margin: 17px 0;
  font-family: "PingFang SC";
  font-weight: 400;

  .sum {
    color: #1f2329;
  }

  .title {
    color: #999999;
    margin-left: 8px;
  }

  .text {
    max-width: 280px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    padding: 1px 22px 1px 6px;
    display: inline-block;
    align-items: center;
    color: #0c296e;
    font-size: 14px;
    line-height: 22px;
    background: rgba(51, 112, 255, 0.1);
    border-radius: 2px;
    margin: 0;
    margin-right: 8px;
    position: relative;
    i {
      position: absolute;
      right: 2px;
      top: 50%;
      transform: translateY(-50%);
      cursor: pointer;
    }
  }

  .clear-btn {
    color: #646a73;
  }

  .clear-btn:hover {
    color: #3370ff;
  }

  .filter-texts-container::-webkit-scrollbar { display: none; }

  .arrow-filter {
    font-size: 16px;
    width: 24px;
    height: 24px;
    cursor: pointer;
    color: #646A73;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .arrow-filter:hover {
    background: rgba(31, 35, 41, 0.1);
    border-radius: 4px;
  }

  .el-icon-arrow-right.arrow-filter {
    margin-left: 5px;
  }

  .el-icon-arrow-left.arrow-filter {
    margin-right: 5px;
  }
  .filter-texts-container {
    flex: 1;
    overflow-x: auto;
    white-space: nowrap;
    height: 24px;
  }
}
.top-operate {
  margin-bottom: 16px;
  .right-user {
    text-align: right;
    display: flex;
    align-items: center;
    justify-content: flex-end;

    .de-button {
      margin-left: 12px;
    }

    .el-input--medium .el-input__icon {
      line-height: 32px;
    }
  }

  .name-email-search {
    width: 240px;
  }
}
</style>
<style lang="scss">
.list-colums-slect {
  padding: 8px 11px !important;
  width: 238px;

  .title,
  .el-checkbox {
    font-family: PingFang SC;
    font-size: 14px;
    font-weight: 400;
    padding: 5px 0;
    margin: 0;
    color: #8f959e;
  }

  .el-checkbox {
    color: #1f2329;
    width: 100%;
  }
}
.de-card-dropdown {
  margin-top: 0 !important;
  .popper__arrow {
    display: none !important;
  }
}
.de-status {
  position: relative;
  margin-left: 15px;
  &::before {
    content: "";
    position: absolute;
    top: 50%;
    left: -13px;
    transform: translateY(-50%);
    width: 5px;
    height: 5px;
    border-radius: 50%;
  }
}

.de-Completed {
  &::before {
    background: var(--primary, #3370ff);
  }
}

.de-Underway {
  &::before {
    background: #8f959e;
  }
}

.de-Pending {
  &::before {
    background: #8f959e;
  }
}

.de-Exec {
  &::before {
    background: var(--primary, #3370ff);
  }
}

.de-Stopped {
  &::before {
    background: var(--deSuccess, #3370ff);
  }
}

.de-Error {
  &::before {
    background: var(--deDanger, #3370ff);
  }

  .el-icon-s-order {
    color: var(--primary, #3370ff);
    cursor: pointer;
  }
}
</style>