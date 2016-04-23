# fluentdLesson

## インストール

`vi /etc/yum.repos.d/td.repo`

```
[treasuredata]
name=TreasureData
baseurl=http://packages.treasure-data.com/redhat/baseurl=http://packages.treasure-data.com/redhat/$basearch
gpgcheck=0
```

`yum install td-agent`


## プロセスの起動等

```
service td-agent start
service td-agent stop
service td-agent restart
service td-agent reload
```

## 各種ファイル・ディレクトリ

設定ファイル  
/etc/td-agent
 
fluentdが出力するログ配置場所  
/var/log/td-agent/

## Apacheのログをfluentdで収集

/var/log/httpd/access_logに追記されると  
/var/log/td-agent/httpd/access.logに  
json形式でアクセスログが収集されるようになる

`vi /etc/td-agent`

末尾に追加 

```
<source>
  type tail  ←in_tailプラグインを指定
  path /var/log/httpd/fffumi.com-access_log  ←アクセスログのパスを指定
  tag apache.access  ←ログに付けるタグを指定
  pos_file /var/log/td-agent/httpd-access_log.pos  ←ファイル内のどの行までを読んだかを記録しておくファイルを指定
  format apache2  ←パースするためにログの書式を指定
</source>

<match apache.access>  ←対象とするタグを指定
  type file  ←out_fileプラグインを指定
  path /var/log/td-agent/httpd/access.log  ←出力先ファイルを指定
  time_slice_format %Y%m%d  ←ファイル名に含める日時情報を指定
  time_slice_wait 10m  ←ログファイルの更新後に旧ログファイルへのログ記録を継続する時間を指定
  compress gzip  ←ログをgzip形式で圧縮
</match>
```

※centosでは/var/log/httpdが700でログが読み込めないので権限変更した
`sudo chmod o+x /var/log/httpd`

## ApacheのログをMongoDBに出力

/var/log/httpd/access_logに追記されると  
mongoDBのfluentdデータベースのapache_accessコレクションに  
10秒おきに登録されるように。

末尾に追加 

```
<match apache.access>
    type mongo
    host localhost
    port 27017
    database fluentd
    collection apache_access
    capped
    capped_size 1024m
    flush_interval 10s
 </match>
```

MongoDBを確認

```
# mongo
> use fluentd
> db.apache_access.find()
```




