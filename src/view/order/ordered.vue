<template>
  <div>
    <Card title="预约管理">
      <Table :columns="columns" :data="orderLists" id="mainTable"></Table>
      <div>
        <div style="margin: 10px 0;overflow: hidden">
          <div style="float: right;">
            <Page size="small" style="height: 20px;" :total="total" :current="current" @on-change="changePage"></Page>
          </div>
        </div>
      </div>
    </Card>
    <Modal
      v-model="showModal"
      title="确认取消"
      @on-ok="onOk">
      <p>请与客户联系后确认取消预约！</p>
    </Modal>
  </div>
</template>

<script>
export default {
  name: 'orderedIndex',
  mounted () {
    this.getOrdersList()
  },
  data () {
    return {
      currentIndex: -1,
      showModal: false,
      total: 0,
      current: 0,
      subscribe_state_map: {
        PENDING: '待处理',
        ACTIVE_CANCEL: '用户主动取消',
        NEGOTIATED_CANCEL: '协商取消',
        GENERATED_ORDER: '已生成订单'
      },
      columns: [
        {
          title: '预约分类',
          key: 'cate_name'
        },
        {
          title: '预约手机号码',
          key: 'subscribe_phone'
        },
        {
          title: '提交预约时间',
          key: 'sub_at'
        },
        {
          title: '状态',
          key: 'subscribe_state',
          render: (h, params) => {
            return h('strong', this.subscribe_state_map[params.row['subscribe_state']])
          }
        },
        {
          title: '操作',
          key: 'order_state_msg',
          render: (h, params) => {
            return h('div',  [
              h('Button', {
                props: {
                  type: 'primary',
                  size: 'small'
                },
                style: {
                  marginRight: '5px',
                  display: params.row.subscribe_state === 'PENDING' ? 'inline': 'none'
                },
                on: {
                  click: () => {
                    this.currentIndex = params.index
                    this.showModal = true
                  }
                }
              }, '取消预约'),
              h('Button', {
                props: {
                  type: 'success',
                  size: 'small'
                },
                style: {
                  display: params.row.subscribe_state === 'PENDING' ? 'inline': 'none'
                },
                on: {
                  click: () => {
                    this.$router.push({
                      name: 'orderGenerate',
                      query: {
                        id: params.row.id,
                        order_sn:params.row.belong_order_sn
                      }
                    })
                  }
                }
              }, '生成订单'),
              h('Button', {
                props: {
                  type: 'error',
                  size: 'small'
                },
                style: {
                  display: params.row.subscribe_state === 'PENDING' ? 'none': 'inline'
                },
                on: {
                  click: () => {
                    this.handleRemove(params.row['belong_order_sn'], params.index)
                  }
                }
              }, '删除')
            ])
          }
        }
      ],
      orderLists: []
    }
  },
  methods: {
    getOrdersList (page = 1) {
      this.$http('user.subscribe.list', {
        rows: 30,
        page
      }).then(rs => {
        this.orderLists = rs.data
        this.total = Number(rs.count)
        this.current = rs.append.page.current
      })
    },
    changePage () {
      this.current = page
      this.getOrdersList(this.current)
    },
    onOk () {
      if (this.currentIndex > -1) {
        if (this.orderLists[this.currentIndex]) {
          const id = this.orderLists[this.currentIndex].id
          this.$http(`user.subscribe.cancel-${id}`).then(() => {
            this.orderLists[this.currentIndex].subscribe_state = 'NEGOTIATED_CANCEL'
          })
        }
      }
    },
    generatingOrder (index) {

    },
    handleRemove (orderSn, rowIndex) {
      if (orderSn) {
        this.$http(`subscribe.delete-${orderSn}`).then(rs => {
          this.getOrdersList()
        })
      }
    }
  }
}
</script>

<style>
  #mainTable .ivu-table-cell,
  #mainTable td
  {
    padding: 1px !important;
    height: 25px;
    line-height: 25px;
    text-align: center;
  }
  #mainTable th {
    height: 25px;
    line-height: 25px;
    text-align: center;
  }
</style>
