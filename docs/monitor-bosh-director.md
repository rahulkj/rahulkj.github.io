How to monitor Bosh Director?
---

You are here, coz you are wanting to know how to monitor your open source bosh director instance/s.

As we know, the bosh create-env command, bootstraps the bosh director for us. During this process, it generates a random password for the using `mbus`, and stores the password in the file, specified in the `--vars-store` variable

Now, there is no direct API exposed by the director on port `25555`, that could be used to pull the director vm statistics.

Inorder to fetch the director vm vitals, processes running on the director vm, releases and the versions running on it, we will need to query the director agent on port `6868`

So let's gather all the information required before we make a `curl` call to the director agent.

Fetch the `mbus_bootstrap_password` from your `bosh-vars.yml` file, by executing `bosh int --path /mbus_bootstrap_password bosh-vars.yml`

Get the certificate used to deploy bosh director, `bosh int --path /default_ca/ca bosh-vars.yml > director-ca.pem`

Now, execute the command `curl -s --cacert director-ca.pem https://mbus:$MBUS_BOOTSTRAP_PASSWORD@$BOSH_DIRECTOR_IP:6868/agent -X POST -d '{"method":"get_state","arguments":["full"],"reply_to":"someone"}' > response.json`

```
{"value":{"properties":{"logging":{"max_log_file_size":""}},"job":{"name":"bosh","release":"","template":"","version":"","templates":[{"name":"bpm","version":"f18421d8c21c425e94bdb3b2df00f3eca2daef29"},{"name":"nats","version":"4d19655090e879ab051dbe10ad350d9d736bae7762e1bd0b31333d8a5f9abbdd"},{"name":"postgres-10","version":"beeba47a8002a140ffae57f585fcd7e1d7c4115de30a3411d6fa6d0f9cba61cc"},{"name":"blobstore","version":"d173497b8ccf11ed85b291c4cc7433650cdc6c164e006beebeb94002ddc14af1"},{"name":"director","version":"d376265d056e50ea0b38d3c5e137b27884a173952c2fc3275a5868944687be22"},{"name":"health_monitor","version":"2dd184a470bd4fc6859a934014eeda921fd656d36bd8a7ed28406158768751e3"},{"name":"vsphere_cpi","version":"d3c6c30b4f012f21f4e3bf3e3fefc56bbf73ce96"},{"name":"user_add","version":"fbcb96cf4e6f1c5467b4d4f84e52e601773736ed"},{"name":"uaa","version":"fa97aa69c86bb94eb621e42bf2e2481f99c572fe"},{"name":"credhub","version":"6fefdd307aebced9af62143bf30fa1c725e6cde3"},{"name":"bbr-credhubdb","version":"32f8192423daa414880d6979a0af8d50dd061926"},{"name":"syslog_forwarder","version":"d12a2f615401b318240d618287582511dd46d8fc"}]},"packages":{"blackbox":{"name":"blackbox","version":"74056b9e5346ca2c77539390e0fd45c867569be2","sha1":"27eb0efe1118747573b8b8531432d9fd62224537","blobstore_id":"0f69a5a2-4551-41c6-75c8-f94a415143ce"},"bosh-gcscli":{"name":"bosh-gcscli","version":"a03b1fc29fd357b8e3023193c87ae9ee49db052965efe5b3d9bff3538ed2df4b","sha1":"sha256:8f3c4e70a2f8a39edf415229af16edb5571ff11657ffc64c46562fd475523977","blobstore_id":"e2af67da-4bd2-4b21-6f59-a140b4c9113d"},"bpm":{"name":"bpm","version":"810ea434b2bb983be369d850ce124a354cd3c69b","sha1":"80d52191d4d5436a88ab9429027f5a9b2f42280e","blobstore_id":"497e1c4f-c768-4f73-5c34-c191bb3d7b84"},"bpm-runc":{"name":"bpm-runc","version":"1e5cc9ab8f23ea7f64a136aa7c4fd67c18effd46","sha1":"5e039262f93d0bec62c83c59e26759c8e0fe77b7","blobstore_id":"116984e0-2ecc-4d1d-5481-88edcdcd021f"},"credhub":{"name":"credhub","version":"8c35e7b56c7adb5cd72b6dfe7ebb5d6bde0eb88f","sha1":"00307481afed8748d3d6b6afc46a8c33a986fd9c","blobstore_id":"dc042e3f-a289-41aa-52cf-8294e05c9c8c"},"davcli":{"name":"davcli","version":"58f558960854f58c55e3d506d3906019178dbc189fbbed1616b8b3c7c02142ea","sha1":"sha256:8927d8dc8dacecfd6f22e80ae4e688330631a23fb96e9d8ad10dc7a3a71bb409","blobstore_id":"822c82a7-89c5-4aa4-69d8-8b0cb00dccc3"},"director":{"name":"director","version":"ef4bfa2fd0cc9839a6bc8cf16992a11b8108f2aa7340eb39642de1278ed714c3","sha1":"sha256:38070bd9efa3b107e2e170c0b91cbeb28378f7c07791db29d7f0f88e9815949e","blobstore_id":"243caa00-8e82-4aef-4bc9-9f112ed87611"},"golang":{"name":"golang","version":"5ee67e992569fbef2bfd303709cc345dfaa71d29","sha1":"b7db3a7fe8a05c88c5ff045b0a279218874e68e8","blobstore_id":"5fb2cc98-31a3-4375-44b8-5d9844cb74cd"},"gonats":{"name":"gonats","version":"f9d600fd5763a72f9268bb081b79658a32d12e6920e8a41905bdeb78a15797e6","sha1":"sha256:1006cd899453dfcf05f702f327bedcf4dbb9365d13d4c9fad24afa889f5e15fe","blobstore_id":"961b9559-02b8-4786-49a1-28924f6a7e3c"},"health_monitor":{"name":"health_monitor","version":"b6b32185ee13f3ceedb48298108a298916b9fa449412582d58edb581c50ec365","sha1":"sha256:6e5edd4cef8b1b37fef931187c04ec4e5fe692f997b74cf4e808191dae69e946","blobstore_id":"dac37dfe-d2d9-482f-5032-f4cff945fb95"},"iso9660wrap":{"name":"iso9660wrap","version":"f9ba5826821d4102b58a14a4b34f524c44155f24","sha1":"05fe3f3d461d8129917f328b80d6d8de2556d5d0","blobstore_id":"8b017eb3-ac6b-40b2-6fd8-4974cc22988d"},"libpq":{"name":"libpq","version":"a83e0662fb3d483552de8e02df46809b6c6755e6f64f83f1cac244db61b5c342","sha1":"sha256:af4e452d313f50cf125f545be78c5ed65b46e26f03ab7dfb40539a1f7ef2c7c6","blobstore_id":"2eb8f7bd-a334-4d7d-58af-27eeca5be03c"},"luna-hsm-client-7.4":{"name":"luna-hsm-client-7.4","version":"746f3c30aadc0af7afc2d5cddcc16d8836a8f845","sha1":"20d449ebaac32a363d654c6795c15ef221398987","blobstore_id":"0e95c15a-7087-47d5-6f4e-4361f21ce9b1"},"mysql":{"name":"mysql","version":"de7350b5d6d835f758a62c0b3a8f07d3f0150d67abfa0da3967bd1127fada4c6","sha1":"sha256:4dc942907510a586258ec1a21c690051b89fcf90b7939ad6f3a45ed7dcff0f1d","blobstore_id":"b7850ce2-f184-4d07-6953-56737ada172e"},"nginx":{"name":"nginx","version":"4cf27fedc69b8b74c26b1e45df7ea715dc9eb427654aab00ef652182b7d81bb5","sha1":"sha256:ec47e0bff555c33b57afe6d4b730ce4395e29ab5ad660a4069457f40af5bd0f6","blobstore_id":"18fc3484-3d5d-4023-7551-8f357a5e99e9"},"openjdk_1.8.0":{"name":"openjdk_1.8.0","version":"d2d85d44c6f0ee050ef1fb0149ce3da575234ad6","sha1":"3ba9062ea0ac748cef6ec4a431493b1f135ba04a","blobstore_id":"54264311-f979-4e4f-66e0-e5a6c7c906a6"},"postgres-10":{"name":"postgres-10","version":"42474d2a099578fd2c4632dca3e90161e2a9e112cc8495ac39be56c7a1792e10","sha1":"sha256:f39607bd7875cd9c89c48129f5f928403cd465e8f9088b12bd4d07eca1cf81f0","blobstore_id":"10c85292-40ba-4c4a-4ba4-dd01621376d5"},"postgres-9.4":{"name":"postgres-9.4","version":"601f3635b43d0e7ba3ae866e3bd69425cdf33f7fb34a7f1bb21cc26818fb598e","sha1":"sha256:744f2e6c03b88fe66422b5f5a6eb3dc860e0002fe1aba60205d968c24f578321","blobstore_id":"3332975a-6827-48cb-5b50-57b4ce9abffa"},"ruby-2.4-r4":{"name":"ruby-2.4-r4","version":"0cdc60ed7fdb326e605479e9275346200af30a25","sha1":"03258435c23cd9de9648fc32ee9883c09eb82ada","blobstore_id":"e45dca34-daef-4543-4109-9679b9d3909e"},"ruby-2.6.3-r0.17.0":{"name":"ruby-2.6.3-r0.17.0","version":"43ece8981ebd9e0979f482c95330e71ce671b211ab6268449466158ba3cbf0a3","sha1":"sha256:165777464245fe9dfbf6cfe28e2e81391b7e32d20c91a0720b81d58f737dd20a","blobstore_id":"5fb05ca3-981b-4d8b-5818-4b98d9aa3c72"},"s3cli":{"name":"s3cli","version":"09abf6d7c9145d3da5e4afc015a19709877be5e043c8dac72f4f82f8110b40d1","sha1":"sha256:4b2719dd3886b676e11bb10382a1feb1f100cbb507bb2acd257872ec3f37af04","blobstore_id":"993f4c78-7f6a-470e-58f0-c8aa99f6d16b"},"tini":{"name":"tini","version":"3d7b02f3eeb480b9581bec4a0096dab9ebdfa4bc","sha1":"e8b9b4499fbec730179cc91ee780a988c69bbbf6","blobstore_id":"e53a0e94-f053-401e-64a5-e4c57d342e9a"},"uaa":{"name":"uaa","version":"65a36da7d5ffff560ffe8908f8b81eeb780e55c7","sha1":"b2a29745b4c66b21204ee343a293e044ad50a88e","blobstore_id":"89174a9e-5b4c-4a14-4aab-965a5c5c3458"},"verify_multidigest":{"name":"verify_multidigest","version":"01334e68c075ea230fa515d32ce1918d0098f707c287a9a76a76a548469ae53d","sha1":"sha256:a798969fd58fabaf4fc933337728bed3c32b0e6f4d0268745127633b1b5ccf8c","blobstore_id":"88aab49c-2189-48c4-47c4-b03e441f542a"},"vsphere_cpi":{"name":"vsphere_cpi","version":"95126fc3d59667323841cc7980c4b061767f7df8","sha1":"05c82f867f4ad0e2c753e1554c4486a010c1f498","blobstore_id":"31821eb1-bfbd-4cd8-6c02-f9f012a7871b"}},"configuration_hash":"unused-configuration-hash","networks":{"default":{"cloud_properties":{"name":"dvPortGroup"},"default":["dns","gateway"],"dns":["10.0.0.1"],"gateway":"10.0.0.1","ip":"10.0.0.31","netmask":"255.255.254.0","type":"manual"}},"resource_pool":{},"deployment":"bosh","name":"bosh","index":0,"id":"0","az":"unknown","persistent_disk":0,"rendered_templates_archive":{"sha1":"sha256:943fecd7957206a622b62e8c6730a0835aba9f5807fc6cbe8fb5fa28da699a9b","blobstore_id":"ca0c57a5-a231-49c9-57e3-468d07a9b084"},"agent_id":"7046a67a-79e9-4519-6dfb-7fe41024905f","job_state":"running","vitals":{"cpu":{"sys":"0.3","user":"0.6","wait":"0.1"},"disk":{"ephemeral":{"inode_percent":"2","percent":"11"},"persistent":{"inode_percent":"0","percent":"6"},"system":{"inode_percent":"34","percent":"50"}},"load":["0.00","0.00","0.00"],"mem":{"kb":"1967504","percent":"49"},"swap":{"kb":"54528","percent":"1"},"uptime":{"secs":11773}},"processes":[{"name":"nats","state":"running","uptime":{"secs":10415},"mem":{"kb":14060,"percent":0.3},"cpu":{"total":0}},{"name":"postgres","state":"running","uptime":{"secs":10414},"mem":{"kb":381432,"percent":9.4},"cpu":{"total":0}},{"name":"blobstore_nginx","state":"running","uptime":{"secs":10413},"mem":{"kb":21252,"percent":0.5},"cpu":{"total":0}},{"name":"director","state":"running","uptime":{"secs":10412},"mem":{"kb":238168,"percent":5.8},"cpu":{"total":0.4}},{"name":"worker_1","state":"running","uptime":{"secs":10412},"mem":{"kb":56356,"percent":1.3},"cpu":{"total":0}},{"name":"worker_2","state":"running","uptime":{"secs":10410},"mem":{"kb":58992,"percent":1.4},"cpu":{"total":0}},{"name":"worker_3","state":"running","uptime":{"secs":10409},"mem":{"kb":59148,"percent":1.4},"cpu":{"total":0}},{"name":"worker_4","state":"running","uptime":{"secs":10408},"mem":{"kb":63164,"percent":1.5},"cpu":{"total":0}},{"name":"director_scheduler","state":"running","uptime":{"secs":10407},"mem":{"kb":58352,"percent":1.4},"cpu":{"total":0}},{"name":"director_sync_dns","state":"running","uptime":{"secs":10406},"mem":{"kb":64412,"percent":1.5},"cpu":{"total":0}},{"name":"director_nginx","state":"running","uptime":{"secs":10405},"mem":{"kb":18028,"percent":0.4},"cpu":{"total":0}},{"name":"health_monitor","state":"running","uptime":{"secs":10404},"mem":{"kb":41136,"percent":1},"cpu":{"total":0}},{"name":"uaa","state":"running","uptime":{"secs":10402},"mem":{"kb":433976,"percent":10.7},"cpu":{"total":0}},{"name":"credhub","state":"running","uptime":{"secs":10370},"mem":{"kb":663420,"percent":16.4},"cpu":{"total":0}},{"name":"blackbox","state":"running","uptime":{"secs":10371},"mem":{"kb":8856,"percent":0.2},"cpu":{"total":0}}],"vm":{"name":"vm-054b7842-eb0f-4f06-b708-c2071355b0b7"}}}
```

Using `jq` cli, you can easily parse the above output, and fetch the director vm resource statistics. `cat response.json | jq '.value.vitals'`

This should output the director vm's vitals:
```
{
  "cpu": {
    "sys": "0.2",
    "user": "0.5",
    "wait": "0.2"
  },
  "disk": {
    "ephemeral": {
      "inode_percent": "2",
      "percent": "9"
    },
    "persistent": {
      "inode_percent": "0",
      "percent": "6"
    },
    "system": {
      "inode_percent": "34",
      "percent": "50"
    }
  },
  "load": [
    "0.00",
    "0.00",
    "0.00"
  ],
  "mem": {
    "kb": "2014648",
    "percent": "50"
  },
  "swap": {
    "kb": "19724",
    "percent": "0"
  },
  "uptime": {
    "secs": 83556
  }
}
```

Similarly, you can get the process level CPU and memory usage. `cat response.json | jq '.value.processes'` which will provide you the info:
```
[
  {
    "name": "nats",
    "state": "running",
    "uptime": {
      "secs": 82336
    },
    "mem": {
      "kb": 15620,
      "percent": 0.3
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "postgres",
    "state": "running",
    "uptime": {
      "secs": 82336
    },
    "mem": {
      "kb": 423836,
      "percent": 10.4
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "blobstore_nginx",
    "state": "running",
    "uptime": {
      "secs": 82335
    },
    "mem": {
      "kb": 21864,
      "percent": 0.5
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "director",
    "state": "running",
    "uptime": {
      "secs": 82334
    },
    "mem": {
      "kb": 266632,
      "percent": 6.5
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "worker_1",
    "state": "running",
    "uptime": {
      "secs": 82333
    },
    "mem": {
      "kb": 59800,
      "percent": 1.4
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "worker_2",
    "state": "running",
    "uptime": {
      "secs": 82332
    },
    "mem": {
      "kb": 63752,
      "percent": 1.5
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "worker_3",
    "state": "running",
    "uptime": {
      "secs": 82331
    },
    "mem": {
      "kb": 63808,
      "percent": 1.5
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "worker_4",
    "state": "running",
    "uptime": {
      "secs": 82330
    },
    "mem": {
      "kb": 63716,
      "percent": 1.5
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "director_scheduler",
    "state": "running",
    "uptime": {
      "secs": 82328
    },
    "mem": {
      "kb": 63500,
      "percent": 1.5
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "director_sync_dns",
    "state": "running",
    "uptime": {
      "secs": 82327
    },
    "mem": {
      "kb": 66544,
      "percent": 1.6
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "director_nginx",
    "state": "running",
    "uptime": {
      "secs": 82326
    },
    "mem": {
      "kb": 21960,
      "percent": 0.5
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "health_monitor",
    "state": "running",
    "uptime": {
      "secs": 82325
    },
    "mem": {
      "kb": 42472,
      "percent": 1
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "uaa",
    "state": "running",
    "uptime": {
      "secs": 82323
    },
    "mem": {
      "kb": 408356,
      "percent": 10.1
    },
    "cpu": {
      "total": 0
    }
  },
  {
    "name": "credhub",
    "state": "running",
    "uptime": {
      "secs": 82292
    },
    "mem": {
      "kb": 676264,
      "percent": 16.7
    },
    "cpu": {
      "total": 0
    }
  }
]
```

You can consume the above data in your logging tool or your monitoring tool.

You can fetch more info from the same output if you wish to. Hope this blog helps you monitor your bosh instance/s.
