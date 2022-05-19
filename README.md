DELETE /mytest

PUT /mytest
{
  "mappings": {
    "properties": {
      "appid": {
        "type": "keyword"
      },
      "deviceid": {
        "type": "keyword"
      },
      "ip": {
        "type": "keyword"
      },
      "servername": {
        "type": "keyword"
      }
    }
  }
}


PUT /mytest/_doc/1
{
"appid": "1",
"deviceid": "2",
"ip": "",
 "servername":"test"
}

PUT /mytest/_doc/2
{
"appid": "1",
"deviceid": "",
"ip": "1.1.1.1",
 "servername":"test"
}

PUT /mytest/_doc/3
{
"appid": "1",
"deviceid": "2",
"ip": "1.1.1.1",
 "servername":""
}

DELETE mytest_transform

POST _transform/mytesttransform/_stop?force=true

DELETE _transform/mytesttransform

PUT _transform/mytesttransform
{
  "source": {
    "index": "mytest"
  },
  "dest": {
    "index": "mytest_transform"
  },
  "pivot": {
    "group_by": {
      "appid": {
        "terms": {
          "field": "appid"
        }
      }
    },
    "aggs": {
      "data": {
        "scripted_metric": {
          "init_script": "state.join = new HashMap()",
          "map_script": "String[] fields = new String[] {'deviceid', 'ip', 'servername'}; for (e in fields) { if (doc.containsKey(e) && doc[e].value!='') {state.join.put(e, doc[e].value)}}",
          "combine_script": "return state.join",
          "reduce_script": "String[] fields = new String[] {'deviceid', 'ip', 'servername'}; Map j=new HashMap(); for (s in states) {for (e in fields) { if (s.containsKey(e) && s[e]!='') {j.put(e, s[e])}}} return j;"
        }
      }
    }
  }
}


POST _transform/mytesttransform/_start

GET mytest_transform/_search

get /mytest/_search
