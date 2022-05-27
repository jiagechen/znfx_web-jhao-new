<template style="padding: 20px">
  <div>

    <!-- 功能区域 -->
    <div style="margin: 10px 0">

      <!-- 去除新增功能，该数据是经过原始数据处理得来，不能够直接导入 或者 新增
      <el-button type="primary" style="margin-left: 5px" @click="addTrack">新增</el-button>
      <el-button type="primary">导入</el-button>
      -->
      <!--
      <el-button type="primary" style="margin-left: 5px" @click="exportData">导出(.xlsx)</el-button>
      <el-button type="primary" @click="exportDataCsv">导出(.csv)</el-button>
      -->
      <el-button type="primary" style="margin-left: 5px" @click="dialogimport = true">导入</el-button>
      <!--<el-button type="primary" style="margin-left: 5px" @click="dialogimportdata = true">数据导入</el-button>-->
    </div>

    <el-table :data="tableData" border style="width: 100%" max-height="500">
      <el-table-column fixed prop="id" label="航迹ID" />
      <el-table-column prop="targetId" label="目标id" />
      <el-table-column prop="handle" label="handle" />
      <el-table-column prop="trackPointItems.length" label="航迹点数量" />
      <el-table-column fixed="right" label="操作">
        <template #default="scope">
          <el-button
            @click="checkStack(scope.row)"
          >查看
          </el-button>
          <!-- 该功能删除，不能够直接添加航迹点，该数据是原始数据经过处理得来的
          <el-button @click="addPoint(scope.row._id)"
          >添加轨迹点</el-button>
          <el-popconfirm title="确定删除吗？" @confirm="handleDelete(scope.row._id)">
              <template #reference>
                  <el-button >删除航线</el-button>
              </template>
          </el-popconfirm>
          -->
        </template>
      </el-table-column>
    </el-table>

    <div style="margin: 10px 0px">
      <el-pagination
        :background="true"
        :current-page="currentPage"
        :page-sizes="[5, 10, 20]"
        :page-size="5"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
    <el-dialog
      :visible.sync="dialogVisible"
      title="航迹点信息"
      width="80%"
      :before-close="handleClose"
    >
      <el-table :data="trackData" border style="width: 100%" height="250">
        <el-table-column fixed prop="itemid" label="ID" />
        <el-table-column prop="latitude" label="维度" />
        <el-table-column prop="longitude" label="经度" />
        <el-table-column prop="height" label="高度" />
        <el-table-column prop="heading" label="方向" />
        <el-table-column prop="speed" label="速度" />
        <el-table-column prop="time_stamp" label="时间戳" />
        <!--
        <el-table-column fixed="right"  label="操作" >
            <template #default="scope">
                <el-button @click="handleEdit(scope.row)">修改</el-button>
                <el-popconfirm title="确定删除吗？" @confirm="deletePoint(trackid,scope.row.itemid)">
                    <template #reference>
                        <el-button >删除</el-button>
                    </template>
                </el-popconfirm>
            </template>
        </el-table-column>
        -->
      </el-table>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
        </span>
      </template>
    </el-dialog>
    <el-dialog
      :visible.sync="dialogimport"
      title="航迹点信息"
      width="80%"
      :before-close="handleClose"
    >
      <!-- 上传组件 -->
      <div v-loading="loading" align="center" element-loading-text="文件导入完毕，正在数据处理中，请稍等">
        <el-upload
          ref="upload"
          class="upload-demo"
          drag
          :limit="1"
          :action="actionUrl"
          :on-exceed="handleExceed"
          :http-request="handUpLoad"
          :auto-upload="false"
        >
          <i class="el-icon-upload" />
          <div class="el-upload__text">将文件拖到此处，或<em>点击上传，仅支持txt/csv格式</em></div>
        </el-upload>
        <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
        <!--<div>
          <span>{{ dataMsg }}</span>
        </div>-->
        <!-- 进度显示 -->
        <br><br>
        <div class="progress-box" style="width: 300px">
          <span>上传进度：</span>
          <el-progress :text-inside="true" :stroke-width="20" :percentage="percent" status="success" />
        </div>
        <br><br><br>
        <div>
          <el-row :gutter="20">
            <span>关键信息所在列,没有标记为-1(-1 <= 所在列 < 总列数)</span><br>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="6"><div class="grid-content bg-purple">
              targetId：<el-input-number v-model="dataForm.targetId" :min="-1" :max="dataForm.total - 1" label="描述文字" @change="handleChange" />
            </div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple">
              longitude：<el-input-number v-model="dataForm.longitude" :min="-1" :max="dataForm.total - 1" label="描述文字" @change="handleChange" />
            </div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple">
              latitude：<el-input-number v-model="dataForm.latitude" :min="-1" :max="dataForm.total - 1" label="描述文字" @change="handleChange" />
            </div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple">
              height：<el-input-number v-model="dataForm.height" :min="-1" :max="dataForm.total - 1" label="描述文字" @change="handleChange" />
            </div></el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="6"><div class="grid-content bg-purple">
              timeStamp：<el-input-number v-model="dataForm.timeStamp" :min="-1" :max="dataForm.total - 1" label="描述文字" @change="handleChange" />
            </div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple">
              heading：<el-input-number v-model="dataForm.heading" :min="-1" :max="dataForm.total - 1" label="描述文字" @change="handleChange" />
            </div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple">
              speed：<el-input-number v-model="dataForm.speed" :min="-1" :max="dataForm.total - 1" label="描述文字" @change="handleChange" />
            </div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple">
              总列数：<el-input-number v-model="dataForm.total" :min="0" :max="50" label="描述文字" @change="handleChange" />
            </div></el-col>
          </el-row>
        </div>
        <!--<el-progress :percentage="100" :format="format"></el-progress>-->
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogimport = false">取 消</el-button>
          <el-button type="primary" @click="dialogimport = false">确 定</el-button>
        </span>
      </template>
    </el-dialog>

  </div>
</template>

<script>
// @ is an alias to /src
import request from '@/utils/request'

export default {
  name: 'TrackTargetAviation',
  components: {},
  data() {
    return {
      loading: false, // 文件导入后，开始数据处理时loading = true，数据处理完毕loading = false
      actionUrl: 'dev/api/tracktarget/uploadaviation', // 上传的后台地址
      shardSize: 5 * 1024 * 1024, // 文件分片大小 5M
      dialogVisible: false,
      dialogimport: false,
      dataMsg: '未选择文件', // 文件上传的文字提示
      acceptType: '.txt,.csv', // 支持的文件类型
      dataProcessState: 0, // 0：未选择文件 1：数据处理中，请等待 2：数据处理完毕，导入中 3：数据导入完毕
      percent: 0, // 上传百分比，如：60%
      fileType: 'txt',
      percentCount: 0,
      totalCnt: 0, // 总共文件分的份数,
      sentCnt: 0, // 文件已经发送的分数
      chunkList: [],
      trackData: [],
      currentPage: 1,
      pageSize: 5,
      total: 10,
      tableData: [],
      row: [], // 用于回调，标识scope.row
      form: {
        id: '',
        targetId: '',
        handle: '',
        trackPointItems: []
      },
      trackForm: {
        itemid: '',
        longitude: '',
        latitude: '',
        height: '',
        heading: '',
        speed: '',
        time_stamp: ''
      },
      dataForm: {
        targetId: 1,
        longitude: 2,
        latitude: 3,
        height: 6,
        timeStamp: 0,
        heading: 5,
        speed: 4,
        total: 16
      }
    }
  },
  created() { // 刚进入时调用
    this.load()
  },
  methods: {
    handleChange() {
      // 如果有关键字段在同一列，就警告一下
      var arr = []
      arr.push(this.dataForm.targetId)
      arr.push(this.dataForm.longitude)
      arr.push(this.dataForm.latitude)
      arr.push(this.dataForm.height)
      arr.push(this.dataForm.timeStamp)
      arr.push(this.dataForm.heading)
      arr.push(this.dataForm.speed)
      arr.push(this.dataForm.total)
      var hash = {}
      for (var i in arr) {
        // eslint-disable-next-line no-empty
        if (hash[arr[i]]) { // 重复，警告一下
          this.$message({
            message: '警告，存在不同字段使用重复列！',
            type: 'warning'
          })
        }
        hash[arr[i]] = true
      }
    },
    load() {
      request.get('api/tracktarget/aviation', {
        params: {
          pageNum: this.currentPage,
          pageSize: this.pageSize
        }
      }).then(res => {
        console.log(res)
        this.tableData = res.data.list
        this.total = res.data.total
      })
    },
    checkStack(row) {
      console.log('row:', row)
      this.trackData = row.trackPointItems
      this.dialogVisible = true
      this.row = row
    },
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      this.load()
    },
    handleCurrentChange(pageNum) {
      this.currentPage = pageNum
      this.load()
    },
    handleClose(done) {
      this.$confirm('确认关闭？')
        .then(_ => {
          done()
        })
        .catch(_ => {})
    },
    handleExceed(files, fileList) { // 从此往下是新的文件上传
      this.$message.warning(`当前限制选择 1个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`)
    },
    submitUpload() {
      this.$refs.upload.submit()
    },
    async check(key) {
      var res = await this.$http.get('dev/api/tracktarget/checkaviation', {
        params: { 'key': key }
      })
      const resData = res.data
      return resData.data
    },
    async recursionUpload(param, file) {
      // FormData私有类对象，访问不到，可以通过get判断值是否传进去
      const _this = this
      const key = param.key
      const shardIndex = param.shardIndex
      const shardTotal = param.shardTotal // 分片总数
      const shardSize = param.shardSize
      const size = param.size
      const fileName = param.fileName
      const suffix = param.suffix
      const fileShard = _this.getFileShard(shardIndex, shardSize, file)
      // param.append("file", fileShard);//文件切分后的分片
      // param.file = fileShard;
      const totalParam = new FormData()
      totalParam.append('file', fileShard)
      totalParam.append('key', key)
      totalParam.append('shardIndex', shardIndex)
      totalParam.append('shardSize', shardSize)
      totalParam.append('shardTotal', shardTotal)
      totalParam.append('size', size)
      totalParam.append('fileName', fileName)
      totalParam.append('suffix', suffix)
      totalParam.append('targetId', this.dataForm.targetId)
      totalParam.append('longitude', this.dataForm.longitude)
      totalParam.append('latitude', this.dataForm.latitude)
      totalParam.append('height', this.dataForm.height)
      totalParam.append('timeStamp', this.dataForm.timeStamp)
      totalParam.append('heading', this.dataForm.heading)
      totalParam.append('speed', this.dataForm.speed)
      totalParam.append('total', this.dataForm.total)
      const config = {
        // 添加请求头
        headers: { 'Content-Type': 'multipart/form-data' }
      }
      console.log(param)
      var res = await this.$http.post('dev/api/tracktarget/uploadaviation', totalParam, config)
      var resData = res.data
      console.log(res)
      // eslint-disable-next-line eqeqeq
      if (resData.code == 200) {
        if (shardIndex < shardTotal) {
          // eslint-disable-next-line eqeqeq
          if (shardTotal != 0) {
            this.percent = 100 * shardIndex / shardTotal
          }
          // eslint-disable-next-line eqeqeq
          if (shardIndex == shardTotal - 1) {
            this.loading = true
          }
          console.log('percent', this.percent)
          console.log('loading', this.loading)
          this.$notify({
            title: '成功',
            message: '分片' + shardIndex + '上传完成。。。。。。',
            type: 'success'
          })
        } else {
          this.loading = false
          this.percent = 100
          this.$notify({
            title: '全部成功',
            message: '文件上传完成。。。。。。',
            type: 'success'
          })
          this.load() // 刷新
        }
        if (shardIndex < shardTotal) {
          console.log('下一份片开始。。。。。。')
          // 上传下一个分片
          // eslint-disable-next-line require-atomic-updates
          param.shardIndex = param.shardIndex + 1
          _this.recursionUpload(param, file)
        }
      }
    },
    async handUpLoad(req) {
      const _this = this
      var file = req.file
      /*  console.log('handUpLoad', req)
                console.log(file);*/
      // let param = new FormData();
      // 通过append向form对象添加数据
      // 文件名称和格式，方便后台合并的时候知道要合成什么格式
      const fileName = file.name
      const suffix = fileName.substring(fileName.lastIndexOf('.') + 1, fileName.length).toLowerCase()
      // 这里判断文件格式，有其他格式的自行判断
      // eslint-disable-next-line eqeqeq
      if (suffix != 'txt' && suffix != 'csv') {
        this.$message.error('文件格式错误')
        return
      }
      // 文件分片
      // let shardSize = 10 * 1024 * 1024;    //以10MB为一个分片
      // let shardSize = 50 * 1024;    //以50KB为一个分片
      const shardSize = _this.shardSize
      const shardIndex = 1		// 分片索引，1表示第1个分片
      const size = file.size
      const shardTotal = Math.ceil(size / shardSize) // 总片数
      // 生成文件标识，标识多次上传的是不是同一个文件
      const key = this.$md5(file.name + file.size + file.type)
      const param = {
        key: key,
        shardIndex: shardIndex,
        shardSize: shardSize,
        shardTotal: shardTotal,
        size: size,
        fileName: fileName,
        suffix: suffix
      }
      /* param.append("uid", key);
              param.append("shardIndex", shardIndex);
              param.append("shardSize", shardSize);
              param.append("shardTotal", shardTotal);
              param.append("size", size);
              param.append("fileName", fileName);
              param.append("suffix", suffix);
          */
      const checkIndexData = await _this.check(key)// 得到文件分片索引
      const checkIndex = checkIndexData.findex

      // if(checkIndex )
      // console.log(checkIndexData)
      // eslint-disable-next-line eqeqeq
      if (checkIndex == -1) {
        this.recursionUpload(param, file)
      } else if (checkIndex < shardTotal) {
        param.shardIndex = param.shardIndex + 1
        this.recursionUpload(param, file)
      } else {
        // this.videoUrl = checkIndexData.fname// 把地址赋值给视频标签
        this.loading = false // 文件导入、数据处理完毕
        this.percent = 100 // 文件导入完毕，进度条100%
        this.$message({
          message: '极速秒传成功。。。。。',
          type: 'success'
        })
        // 数据导入完毕后，刷新页面
        this.load()
      }
      // console.log('结果：', res)
    },
    getFileShard(shardIndex, shardSize, file) {
      // eslint-disable-next-line no-unused-vars
      const _this = this
      const start = (shardIndex - 1) * shardSize	// 当前分片起始位置
      const end = Math.min(file.size, start + shardSize) // 当前分片结束位置
      const fileShard = file.slice(start, end) // 从文件中截取当前的分片数据
      return fileShard
    }
  }
}
</script>
<style>
  .el-row {
    margin-bottom: 20px;
  &:last-child {
     margin-bottom: 0;
   }
  }
  .el-col {
    border-radius: 4px;
  }
  .bg-purple-dark {
    background: #99a9bf;
  }
  .bg-purple {
    background: #d3dce6;
  }
  .bg-purple-light {
    background: #e5e9f2;
  }
  .grid-content {
    border-radius: 4px;
    min-height: 36px;
  }
  .row-bg {
    padding: 10px 0;
    background-color: #f9fafc;
  }
</style>
