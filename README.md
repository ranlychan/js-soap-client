Javascript SOAP Client
======================

Forked from https://github.com/gotshub/js-soap-client and modified by ranlychan.
分叉自 https://github.com/gotshub/js-soap-client 由ranlychan修改。

改进Improvements
------------

* Adapted to Vue 3.0 environment with webpack
* 适配到使用webpack的Vue 3.0环境
* Export SOAPClientParameters, SOAPClient for importing
* 使用export将SOAPClientParameters, SOAPClient导出以方便在其它位置使用import导入和使用
* Fixed some prasing problem
* 修复了一些数据解析问题

测试Tested
------

* Tested with my own WebService
* 使用我自行发布的WebService解析测试
* Self defined object array has been put into use
* WebService接口返回带有自定义的类
* This soap client is used as a util in vue-element-admin
* 这个soapclient.js预期在vue-element-admin模板下作为一个util工作

使用Using
------
下面是在vue-element-admin下进一步封装请求和调用的示例。
![demo.png](https://s1.328888.xyz/2022/06/06/zuO3A.png)

soap-request.js

```JavaScript
import { SOAPClientParameters, SOAPClient } from '@/utils/soapclient'
import Vue from 'vue'

export function requestSOAP(paramsKV) {
  var parameters = new SOAPClientParameters()
  var query = paramsKV['params']
  var url = paramsKV['url']
  var method = paramsKV['method']
  var callback = paramsKV['callback']

  if (query != null && Object.keys(query).length > 0) {
    Object.keys(query).forEach(function(key) {
      parameters.add(key, query[key])
    })
  }
  SOAPClient.invoke(url, method, parameters, true, function(response, xmlResponse) {
    console.log(xmlResponse)

    const ob_response = Vue.observable(response)
    var code
    var data
    if (response instanceof Error) {
      code = response.code
      data = response.message
    } else {
      code = 200
      data = { items: ob_response, total: response.length }
    }
    const r = {
      code: code,
      data: data
    }
    callback(r)
  })
}

```

my-api.js

```JavaScript
import { requestSOAP } from '@/utils/soap-request'
// 上面导入请求封装中间件

/**
 * 请求血压数据列表
 * @param query
 * @param callback
 */
export function getBloodPressure(query, callback) {
  return requestSOAP({
    url: 'http://localhost:8080/MyWebService/services/BloodPressureService',
    method: 'getBloodPressureList',
    params: query,
    callback: callback
  })
}

/**
 * 删除指定bpId血压数据记录
 * @param query ==> {'bpId':id}
 * @param callback
 */
export function deleteBloodPressure(query, callback) {
  return requestSOAP({
    url: 'http://localhost:8080/MyWebService/services/BloodPressureService',
    method: 'deleteBloodPressure',
    params: query,
    callback: callback
  })
}
```

mypage.vue
```JavaScript
<template>
  <div class="app-container">
    <el-table v-loading="listLoading" :data="list" border fit highlight-current-row style="width: 100%">
      <el-table-column align="center" label="记录ID" width="100%">
        <template slot-scope="scope">
          <span>{{ scope.row.bpId }}</span>
        </template>
      </el-table-column>

      <el-table-column width="180px" align="center" label="检测时间">
        <template slot-scope="scope">
          <span>{{ scope.row.upgradeTime | parseTime('{y}/{m}/{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>

      <el-table-column class-name="status-col" label="血压状态" width="110">
        <template slot-scope="{row}">
          <el-tag :type="row.status | statusFilter">
            {{ row.status }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column width="120px" align="center" label="收缩压">
        <template slot-scope="scope">
          <span>{{ scope.row.sbpValue }}</span>
        </template>
      </el-table-column>

      <el-table-column width="120px" align="center" label="舒张压">
        <template slot-scope="scope">
          <span>{{ scope.row.dbpValue }}</span>
        </template>
      </el-table-column>

      <el-table-column width="120px" align="center" label="姓名">
        <template slot-scope="scope">
          <span>{{ scope.row.realName }}</span>
        </template>
      </el-table-column>

      <el-table-column width="120px" align="center" label="收缩压上限">
        <template slot-scope="scope">
          <span>{{ scope.row.sbpMax }}</span>
        </template>
      </el-table-column>

      <el-table-column width="120px" align="center" label="收缩压下限">
        <template slot-scope="scope">
          <span>{{ scope.row.sbpMin }}</span>
        </template>
      </el-table-column>

      <el-table-column width="120px" align="center" label="舒张压上限">
        <template slot-scope="scope">
          <span>{{ scope.row.dbpMax }}</span>
        </template>
      </el-table-column>

      <el-table-column width="120px" align="center" label="舒张压下限">
        <template slot-scope="scope">
          <span>{{ scope.row.dbpMin }}</span>
        </template>
      </el-table-column>
      
      <el-table-column align="center" label="操作" width="120">
        <template slot-scope="scope">
          <el-button type="primary" size="small" icon="el-icon-delete" v-on:click="deleteBp">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />
  </div>
</template>

<script>
import { deleteBloodPressure, getBloodPressure } from '@/api/patient'

import Pagination from '@/components/Pagination' // Secondary package based on el-pagination

export default {
  name: 'QueryPatientData',
  components: { Pagination },
  filters: {
    statusFilter(status) {
      const statusMap = {
        normal: 'success',
        danger: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20
      }
    }
  },
  created() {
    // 页面创建后自动调用请求方法
    this.getList()
  },
  methods: {
    getList() {
      // 请求血压记录列表数据并存入this.list
      this.listLoading = true
      getBloodPressure({}, response => {
        console.log(response)
        console.log(typeof response)

        this.list = response.data.items
        this.total = response.data.total
        this.listLoading = false
      })
    }
    // 删除血压记录方法，不一定调用
    ,deleteBp() {
      deleteBloodPressure({ 'bpId': 3 }, response => {
        console.log(response)
        console.log(typeof response)
      })
    }
  }
}
</script>

<style scoped>
.edit-input {
  padding-right: 100px;
}
.cancel-btn {
  position: absolute;
  right: 15px;
  top: 10px;
}
</style>

```


