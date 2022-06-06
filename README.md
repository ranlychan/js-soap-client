Javascript SOAP Client
======================

Forked from https://github.com/gotshub/js-soap-client and modified by ranlychan.

Improvements
------------

* Adapted to Vue 3.0 environment with webpack
* Export SOAPClientParameters, SOAPClient for importing
* Fixed some prasing problem

Tested
------

* Tested with my own WebService
* Self defined object array has been put into use
* This soap client is used as a util in vue-element-admin


Using
------
soap-request.js

```JavaScript
import { SOAPClientParameters, SOAPClient } from '@/utils/soap-client-origin'
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
```
