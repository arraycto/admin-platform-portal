<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :md="6" :sm="12">
            <a-form-item label="账号">
              <!--<a-input placeholder="请输入账号查询" v-model="queryParam.username"></a-input>-->
              <h-input placeholder="输入账号模糊查询" v-model="queryParam.username"></h-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <a-form-item label="性别">
              <a-select v-model="queryParam.sex" placeholder="请选择性别">
                <a-select-option value="">请选择</a-select-option>
                <a-select-option value="1">男性</a-select-option>
                <a-select-option value="2">女性</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>


          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="真实名字">
                <a-input placeholder="请输入真实名字" v-model="queryParam.realname"></a-input>
              </a-form-item>
            </a-col>

            <a-col :md="6" :sm="8">
              <a-form-item label="手机号码">
                <a-input placeholder="请输入手机号码查询" v-model="queryParam.phone"></a-input>
              </a-form-item>
            </a-col>

            <a-col :md="6" :sm="8">
              <a-form-item label="用户状态">
                <a-select v-model="queryParam.status" placeholder="请选择">
                  <a-select-option value="">请选择</a-select-option>
                  <a-select-option value="1">正常</a-select-option>
                  <a-select-option value="2">冻结</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </template>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator" style="border-top: 5px">
      <a-button @click="handleAdd" type="primary" icon="plus">添加用户</a-button>
      <a-button @click="handleSyncUser" v-has="'user:syncbpm'" type="primary" icon="plus">同步流程</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('用户信息')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-button type="primary" icon="hdd" @click="recycleBinVisible=true">回收站</a-button>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay" @click="handleMenuClick">
          <a-menu-item key="1">
            <a-icon type="delete" @click="batchDel"/>
            删除
          </a-menu-item>
          <a-menu-item key="2">
            <a-icon type="lock" @click="batchFrozen('2')"/>
            冻结
          </a-menu-item>
          <a-menu-item key="3">
            <a-icon type="unlock" @click="batchFrozen('1')"/>
            解冻
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i>已选择&nbsp;<a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项&nbsp;&nbsp;
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        bordered
        size="middle"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <template slot="avatarslot" slot-scope="text, record, index">
          <div class="anty-img-wrap">
            <a-avatar shape="square" :src="record.avatar" icon="user"/>
          </div>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical"/>

          <a-dropdown>
            <a class="ant-dropdown-link">
              更多 <a-icon type="down"/>
            </a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a href="javascript:;" @click="handleDetail(record)">详情</a>
              </a-menu-item>

              <a-menu-item>
                <a href="javascript:;" @click="handleChangePassword(record.username)">密码</a>
              </a-menu-item>

              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>

              <a-menu-item v-if="record.status==1">
                <a-popconfirm title="确定冻结吗?" @confirm="() => handleFrozen(record.id,2,record.username)">
                  <a>冻结</a>
                </a-popconfirm>
              </a-menu-item>

              <a-menu-item v-if="record.status==2">
                <a-popconfirm title="确定解冻吗?" @confirm="() => handleFrozen(record.id,1,record.username)">
                  <a>解冻</a>
                </a-popconfirm>
              </a-menu-item>

              <a-menu-item>
                <a href="javascript:;" @click="handleAgentSettings(record.username)">代理人</a>
              </a-menu-item>

            </a-menu>
          </a-dropdown>
        </span>


      </a-table>
    </div>
    <!-- table区域-end -->

    <user-modal ref="modalForm" @ok="modalFormOk"></user-modal>

    <password-modal ref="passwordmodal" @ok="passwordModalOk"></password-modal>

    <!-- 用户回收站 -->
    <user-recycle-bin-modal :visible.sync="recycleBinVisible" @ok="modalFormOk"/>

  </a-card>
</template>

<script>
  import UserModal from './UserModal'
  import PasswordModal from './PasswordModal'
  import UserRecycleBinModal from './UserRecycleBinModal'
  import { ManageMixin } from '@/mixins/ManageMixin'
  import { frozenBatch } from '@/api/api'
  import HInput from '@/components/assem/HInput'

  export default {
    name: 'UserMange',
    mixins: [ManageMixin],
    components: {
      UserModal,
      PasswordModal,
      UserRecycleBinModal,
      HInput
    },
    data () {
      return {
        description: '这是用户管理页面',
        queryParam: {},
        recycleBinVisible: false,
        columns: [
          {
            title: '用户账号',
            align: 'center',
            dataIndex: 'username',
            width: 120
          },
          {
            title: '用户姓名',
            align: 'center',
            width: 100,
            dataIndex: 'realname',
          },
          {
            title: '头像',
            align: 'center',
            width: 120,
            dataIndex: 'avatar',
            scopedSlots: { customRender: 'avatarslot' }
          },

          {
            title: '性别',
            align: 'center',
            width: 80,
            dataIndex: 'sex_dictText',
            sorter: true
          },
          {
            title: '生日',
            align: 'center',
            width: 100,
            dataIndex: 'birthday'
          },
          {
            title: '手机号码',
            align: 'center',
            width: 100,
            dataIndex: 'phone'
          },
          {
            title: '部门',
            align: 'center',
            width: 180,
            dataIndex: 'orgCode'
          },
          {
            title: '状态',
            align: 'center',
            width: 80,
            dataIndex: 'status_dictText'
          },
          {
            title: '操作',
            dataIndex: 'action',
            scopedSlots: { customRender: 'action' },
            align: 'center',
            width: 170
          }

        ],
        url: {
          syncUser: '/api-platf/process/extActProcess/doSyncUser',
          list: '/api-platf/platform/user/list',
          delete: '/api-platf/platform/user/delete',
          deleteBatch: '/api-platf/platform/user/deleteBatch',
          exportXlsUrl: '/api-platf/platform/user/exportXls',
          importExcelUrl: '/api-platf/platform/user/importExcel'
        },
      }
    },
    computed: {
      importExcelUrl: function () {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      }
    },
    methods: {
      async batchFrozen (status) {
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条记录！')
          return false
        } else {
          let ids = ''
          let isAdmin = false
          this.selectionRows.forEach(function (row) {
            if (row.username === 'admin') {
              isAdmin = true
            }
          })
          if (isAdmin) {
            this.$message.warning('管理员账号不允许此操作,请重新选择！')
            return
          }
          this.selectedRowKeys.forEach(function (val) {
            ids += val + ','
          })
          const onOk = await this.$confirm({ title: '确认操作', content: '是否' + (status === 1 ? '解冻' : '冻结') + '选中账号?' })
          if (onOk) {
            const res = await frozenBatch({ ids: ids, status: status })
            if (res.success) {
              this.$message.success(res.message)
              this.loadData()
              this.onClearSelected()
            } else {
              this.$message.warning(res.message)
            }
          }
        }
      },
      handleMenuClick (e) {
        if (e.key === 1) {
          this.batchDel()
        } else if (e.key === 2) {
          this.batchFrozen(2)
        } else if (e.key === 3) {
          this.batchFrozen(1)
        }
      },
      async handleFrozen (id, status, username) {
        //TODO 后台校验管理员角色
        if ('admin' === username) {
          this.$message.warning('管理员账号不允许此操作！')
          return
        }
        const res = await frozenBatch({ ids: id, status: status })
        if (res.success) {
          this.$message.success(res.message)
          this.loadData()
        } else {
          this.$message.warning(res.message)
        }
      },
      handleChangePassword (username) {
        this.$refs.passwordmodal.show(username)
      },
      handleSyncUser () {
      },
      passwordModalOk () {
        //TODO 密码修改完成 不需要刷新页面，可以把datasource中的数据更新一下
      }
    }

  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>