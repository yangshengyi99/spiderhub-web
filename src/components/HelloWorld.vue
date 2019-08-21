<template>
  <div>
    <!--导航条-->
    <a-menu
      v-model="currentTab"
      mode="horizontal"
      theme="dark"
    >
      <a-menu-item>
        爬虫管理
      </a-menu-item>
    </a-menu>
    <br/><br/>
    <!--搜索框-->
    <div id="search">
      <a-input-search
        placeholder="请输入要查找的关键字"
        style="width: 200px"
        @search="onSearch(key_mark)"
        v-model="key_mark"
      />
    </div>
    <!--create新建-->
    <a-button type="primary" @click="showModal">+ 新建</a-button>
    <a-modal
      title="新建"
      cancelText="取消"
      okText="确定"
      :visible="visible"
      @ok="handleSubmit"
      :confirmLoading="confirmLoading"
      @cancel="handleCancel"
    >
      <p>
        <a-form
          :form="form"
        >
          <a-form-item
            label="名字"
            :label-col="{ span: 5 }"
            :wrapper-col="{ span: 12 }"
          >
            <a-input
              v-decorator="[
          'name',
          {rules: [{ required: false }]}
        ]"
            />
          </a-form-item>
          <a-form-item
            label="详情"
            :label-col="{ span: 5 }"
            :wrapper-col="{ span: 12 }"
          >
            <a-input
              v-decorator="[
          'description',
          {rules: [{ required: false }]}
        ]"
            />
          </a-form-item>
          <a-form-item
            label="shm_size"
            :label-col="{ span: 5 }"
            :wrapper-col="{ span: 12 }"
          >
            <a-input
              v-decorator="[
          'params',
          {rules: [{ required: false }]}
        ]"
            />
          </a-form-item>
          <a-form-item
            label="定时启动"
            :label-col="{ span: 5 }"
            :wrapper-col="{ span: 12 }"
          >
            <a-input
              v-decorator="[
          'set_time',
          {rules: [{ required: false }]}
        ]"
            />
          </a-form-item>
          <a-form-item
            label="运行间隔"
            :label-col="{ span: 5 }"
            :wrapper-col="{ span: 12 }"
          >
            <a-input
              v-decorator="[
          'interval',
          {rules: [{ required: false }]}
        ]"
            />
          </a-form-item>
        </a-form>
      </p>
    </a-modal>
    <!--spider列表-->
    <div id="spiderlist">
      <a-table :columns="columns" :dataSource="spiderlist" rowKey="id">
        <span slot="customID">Id</span>
        <span slot="name" slot-scope="record">
          <!--详情-->
           <a-popover title="详情">
             <template slot="content">
              <p>描述：{{record.description}}</p>
              <p>状态：{{record.status}}</p>
              <p>shm_size：{{record.params}}</p>
              <p>运行间隔(s)：{{record.interval}}</p>
              <p>定时启动(h)：{{record.set_time}}</p>
             </template>
              <a href="javascript:;" >{{record.name}}</a>
           </a-popover>
        </span>
        <span slot="created_time" slot-scope="record">
          {{record.created_time|moment}}
        </span>
        <span slot="updated_time" slot-scope="record">
          {{record.updated_time|moment}}
        </span>
        <span slot="status" slot-scope="status,record" >
          <a-badge v-if="status ==='exited'&& setStatus(record.setTime,record.interval)" status="warning" text="待运行" :key="status" >{{statusMap[status]}}</a-badge>
          <a-badge v-else-if="status ==='exited' && !setStatus(record.setTime,record.interval) " status="success" text="已完成" :key="status">{{statusMap[status]}}</a-badge>
          <a-badge v-else-if="status ==='running'" status="error" text="正在运行" :key="status">{{statusMap[status]}}</a-badge>
        </span>
        <span slot="action" slot-scope="record">
          <a href="javascript:;" @click="uploadModal(record.id)">上传</a>
          <a-divider type="vertical"/>
          <a href="javascript:;" @click="showDeleteConfirm(record.id)">删除</a>
          <a-divider type="vertical"/>
          <a href="javascript:;" @click="updateModal(record.id,record.name,record.description,record.params,record.interval,record.set_time)">更新</a>
          <a-divider type="vertical"/>
          <a href="javascript:;" @click="showRunConfirm(record.id,record.params)">运行</a>
          <a-divider type="vertical"/>
          <a href="javascript:;" @click="showStopConfirm(record.id)">停止</a>
        </span>
      </a-table>
      <!--分页-->
      <a-pagination id="pagination"
                    :total="total"
                    :showTotal="total => `总共 ${total} 条数据`"
                    :pageSize="pageSize"
                    v-model="current"
                    @change="changePage"
      >
      </a-pagination>

      <!--update-->
      <a-modal
        title="更新"
        cancelText="取消"
        okText="确定"
        :visible="update"
        :spiderid="spiderid"
        :spiname="spiname"
        :spides="spides"
        :spiparams="spiparams"
        :spiinterval="spiinterval"
        :spisettime="spisettime"
        @ok="handleUpdate(spiderid)"
        :confirmLoading="confirmLoading"
        @cancel="handleUpdateCancel"
      >
        <p>
          <a-form
            :form="form"
          >
            <a-form-item
              label="名字"
              :label-col="{ span: 5 }"
              :wrapper-col="{ span: 12 }"

            >
              <a-input
                v-decorator="[
          'name',
          {rules: [{ required: false }],

      }
          ]"
              />
            </a-form-item>
            <a-form-item
              label="详情"
              :label-col="{ span: 5 }"
              :wrapper-col="{ span: 12 }"
            >
              <a-input :placeholder="spides"
                v-decorator="[
          'description',
          {rules: [{ required: false }]}
        ]"
              />
            </a-form-item>
            <a-form-item
              label="shm_size"
              :label-col="{ span: 5 }"
              :wrapper-col="{ span: 12 }"
            >
              <a-input :placeholder="spiparams"
                v-decorator="[
          'params',
          {rules: [{ required: false }]}
        ]"
              />
            </a-form-item>
            <a-form-item
            label="定时启动"
            :label-col="{ span: 5 }"
            :wrapper-col="{ span: 12 }"
          >
            <a-input :placeholder="spisettime"
              v-decorator="[
          'set_time',
          {rules: [{ required: false }]}
        ]"
            />
          </a-form-item>
          <a-form-item
            label="运行间隔"
            :label-col="{ span: 5 }"
            :wrapper-col="{ span: 12 }"
          >
            <a-input :placeholder="spiinterval"
              v-decorator="[
          'interval',
          {rules: [{ required: false }]}
        ]"
            />
          </a-form-item>
          </a-form>
        </p>
      </a-modal>
      <!--upload-->
      <a-modal
        title="上传"
        cancelText="取消"
        okText="确定"
        :visible="upload"
        :spiderid="spiderid"
        @ok="handleUploadCancle"
        :confirmLoading="confirmLoading"
        @cancel="handleUploadCancle"
      >
        <p>
          <a-upload :action="'http://192.168.124.234:8023/spiders/'+ spiderid + '/workdirectory'" directory
                    :before-upload="beforeUpload">
            <a-button>
              <a-icon type="upload"/>
              上传文件目录
            </a-button>
          </a-upload>

        </p>
      </a-modal>
    </div>
  </div>
</template>
<script>
/* eslint-disable */
  const statusMap = {
    'exited': '',
    'running': '',
  }
  const api = 'http://192.168.124.234:8023/'
  const columns = [
    {
      dataIndex: 'id',
      slots: {title: 'customID'},
      scopedSlots: {customRender: 'id'}
    },
    {
      title: '名字',
      scopedSlots: {customRender: 'name'},
    },

    {
      title: '工作目录',
      dataIndex: 'workspace'
    },
    {
      title: '运行容器',
      dataIndex: 'docker_container'
    },
    {
      title: '创建时间',
      scopedSlots: {customRender: 'created_time'},
    },
    {
      title: '更新时间',
      scopedSlots: {customRender: 'updated_time'},
    },
    {
      title: '状态',
      dataIndex: 'status',
      scopedSlots: {customRender: 'status'}
    },
    {
      title: '相关操作',
      scopedSlots: {customRender: 'action'},
    },
  ];


  export default {
    name: 'HelloWorld',
    data() {
      return {
        spiderlist: [],
        columns,
        formLayout: 'horizontal',
        form: this.$form.createForm(this),
        visible: false,
        confirmLoading: false,
        update: false,
        upload: false,
        total: null,
        current: 1,
        pageSize: 10,
        headers: {
          authorization: 'authorization-text',
        },
        currentTab: ['mail'],
        key_mark: '',
        spiderid: '',
        time: '',
        statusMap: statusMap,
        updated_time_list:'',
        spider_status_list:[],
        spiname:'',
        spides:'',
        spiparams:'',
        spiinterval:'',
        spisettime:'',
        flag:false

      }
    },

    methods: {
      getSpiders: function () {
        this.$http.get(`${api}spiders`).then((res) => {
          this.spiderlist = res.data.data
          this.total = res.data.meta.total
        }).catch(e => {
          console.log(e)
        })
      },
      setStatus:function(setT,setIn){
        if(setT || setIn){
          return true
        }
        else {
          return false
        }
      },

      getStatus: function () {
        this.$http.get(`${api}spiders/status`).then((res) => {
          this.spider_status_list=res.data
          let status = {}
          for(let s of this.spider_status_list){
            status[s['id']]=s['status']
          }
          for(let s of this.spiderlist){
            if (s['id'] in status)
            {
              s['status'] = status[s['id']]
            }
          }
        }).catch(e => {
          console.log(e)
        })

      },
      setTime() {
        let that = this
        setInterval(function () {
          that.getStatus()
        }, 3000)

      },

      deletespiderById: function (spiderid) {
        this.$http.delete(`${api}spiders/${spiderid}`
        ).then((res) => {
          console.log("deleted!")
        }).catch(e => {
          console.log(e)
        })
      },
      showModal() {
        this.visible = true
      },
      handleSubmit(e) {
        e.preventDefault();
        this.form.validateFields((err, values) => {
          if (!err) {
            this.$http.post(`${api}spiders`, values).then((res) => {
              console.log('Submit values of form: ', values);
            }).catch(e => {
              console.log(e)
            })
          }
        });
        this.visible = false;
        this.confirmLoading = false;

      },
      handleCancel(e) {
        console.log('Clicked cancel button');
        this.visible = false
      },
      updateModal(spiderid,spiname,spides,spiparams,spiinterval,spisettime) {
        this.update = true
        this.spiderid = spiderid
        this.spiname=spiname
        this.spides=spides
        this.spiparams=spiparams
        this.spiinterval=spiinterval
        this.spisettime=spisettime
      },
      handleUpdate: function (spiderid) {
        this.form.validateFields((err, values) => {
          if (!err) {
            this.$http.put(`${api}spiders/${spiderid}`, values).then((res) =>{
            }).catch(e => {
              console.log(e)
            })
          }
        });
        this.update = false;
        this.confirmLoading = false;
      },
      handleUpdateCancel(e) {
        console.log('Clicked cancel button');
        this.update = false
      },
      showDeleteConfirm: function (spiderid) {
        const _this = this;
        this.$confirm({
          title: '确定要删除' + spiderid + '吗 ?',
          okText: '确定',
          okType: 'danger',
          cancelText: '取消',
          onOk() {
            _this.deletespiderById(spiderid);
          },
          onCancel() {
            console.log('Cancel');
          },
        });
      },
      uploadModal(spiderid) {
        this.upload = true
        this.spiderid = spiderid
      },
      handleUpload: function (spiderid) {
        this.$http.post(`${api}spiders/${spiderid}/workspace`).then((res) => {
          console.log('Uploaded!');
        }).catch(e => {
          console.log(e)
        })
        this.upload = false;
        this.confirmLoading = false;
      },
      handleUploadCancle(e) {
        console.log('Clicked cancel button');
        this.upload = false
      },

      handleChange(info, spiderid) {
        if (info.file.status !== 'uploading') {
          console.log(info.file, info.fileList);
        }
        if (info.file.status === 'done') {
          this.$message.success(`${info.file.name} file uploaded successfully`);
        } else if (info.file.status === 'error') {
          this.$message.error(`${info.file.name} file upload failed.`);
        }
      },
      changePage: function (current) {
        this.$http.get(`${api}spiders?page=${current}`).then((res) => {
          this.spiderlist = res.data.data
          this.total = res.data.meta.total
        }).catch(e => {
          console.log(e)
        })
      },
      onSearch: function (key_mark) {
        const _this = this;
        _this.$http.get(`${api}spiders/${key_mark}`).then((res) => {
          console.log(res.data)
          _this.spiderlist = res.data.data
          _this.total = res.data.total
          this.setTime()
        }).catch(e => {
          console.log(e)
        })
      },
      beforeUpload: function () {
        const _this = this
        new Promise(function (resolve, reject) {
          _this.$http.delete(`${api}spiders/${_this.spiderid}/beforeupload`).then((res) => {
            console.log(res.data)
            if (res.data == 'ok')
              return resolve
            else {
              return reject
            }
          }).catch(e => {
            console.log(e)
          })
        })
      },
      runspiderById: function (spiderid,runparams) {
        this.flag='True'
        if(runparams){
           this.$http.post(`${api}spiders/${spiderid}/${runparams}/${this.flag}/run`).then((res) => {
          console.log()
        }).catch(e => {
          console.log(e)
        })
        }
        else{
            runparams = 64
            this.$http.post(`${api}spiders/${spiderid}/${runparams}/${this.flag}/run`).then((res) => {
              console.log()
            }).catch(e => {
              console.log(e)
            })
          }

      },
      showRunConfirm: function (spiderid,runparams) {
        const _this = this;
        this.$confirm({
          title: '确定要运行' + spiderid + '吗 ?',
          okText: '确定',
          okType: 'primary',
          cancelText: '取消',
          onOk() {
            _this.runspiderById(spiderid,runparams);
          },
          onCancel() {
            console.log('Cancel');
          },
        });
      },
      stopspiderById: function (spiderid) {
        this.$http.post(`${api}spiders/${spiderid}/stop`).then((res) => {
          console.log(res.data);
        }).catch(e => {
          console.log(e)
        })
      },
      showStopConfirm: function (spiderid) {
        const _this = this;
        this.$confirm({
          title: '确定要停止运行' + spiderid + '吗 ?',
          okText: '确定',
          okType: 'danger',
          cancelText: '取消',
          onOk() {
            _this.stopspiderById(spiderid);
          },
          onCancel() {
            console.log('Cancel');
          },
        });
      },


    },

    mounted: function () {
      this.getSpiders()
      this.setTime()

    }
  }
</script>


<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #pagination {
    position: absolute;
    left: 45%;
  }

  #search {
    float: right;
  }

</style>
