# bsf-avocado

`bsf-avocado` 是一款专为成都极数蓝牙打印SDK定制的UTS插件。

[文档](https://github.com/byteee-fund/bsf-avocado-doc) | [Demo](https://github.com/byteee-fund/bsf-avocado-demo)


## 平台

- Android

## 兼容性
- uniapp
- uniappx


## 使用说明

### 引入插件

```js
import * as Location from "@uni_modules/bsf-avocado";
```

### 初始化SDK

```js
const result = AvocadoSDK.initSDK("9ZWI3PGTRIJE");
```

### 检测权限

```js
if (!AvocadoSDK.checkPermission()) {
    AvocadoSDK.requestPermission({
        onPermit: () => {
            console.log("权限获取成功");
        },
        onRefuse: () => {
            console.log("权限被拒绝");
        }
    });
} else {
    console.log("已具备所需权限");
}
```

### 搜索设备

```js
AvocadoSDK.discover({
    onDiscoveryStarted: () => {
        console.log("开始搜索设备");
    },
    onDeviceFound: (device) => {
        console.log(`发现设备: ${device.name || '未知设备'}`);
    },
    onDiscoveryStopped: () => {
        console.log("停止搜索设备");
    }
});
```

### 停止搜索

```js
AvocadoSDK.stopDiscovery();
```

### 连接设备

```js
AvocadoSDK.connectDevice({
    address: device.address,
    onConnect: (success) => {
        this.isConnected = success;
        console.log(`设备连接${success ? '成功' : '失败'}`);
    }
});
```

### 断开连接

```js
AvocadoSDK.disconnectDevice();
```

### 获取设备状态

```js
AvocadoSDK.getDeviceStatus({
    onResponse: (response) => {
        console.log(`设备状态: ${response}`);
    },
    onErrorResponse: (errorCode) => {
        console.log(`获取设备状态错误: ${errorCode}`);
    }
});
```

### 获取设备信息

```js
AvocadoSDK.getDeviceInfo({
    onResponse: (response) => {
        console.log(`设备信息: ${response}`);
    },
    onErrorResponse: (errorCode) => {
        console.log(`获取设备信息错误: ${errorCode}`);
    }
});
```

### 获取混合状态

```js
AvocadoSDK.getMixedStatus({
    onResponse: (response) => {
        console.log(`混合状态: ${response}`);
    },
    onErrorResponse: (errorCode) => {
        console.log(`获取混合状态错误: ${errorCode}`);
    }
});
```

### 发送打印任务

```js
AvocadoSDK.printJob({
    tempPath: filePath[0].path,
    copies: 1,
    isFirmUpdata: false,
    onCreateJobSuccess: (jobId) => { 
        console.log(`创建打印任务成功，任务ID: ${jobId}`);
    },
    onCreateJobFail: (errorCode, errorReason) => {
        console.log(`创建打印任务失败: ${errorReason}`);
    },
    onProgressChange: (max, current, jobId) => {
        console.log(`打印进度: ${current}/${max}`);
    },
    onTransferCompleted: (jobId) => {
        console.log(`任务${jobId}传输完成`);
    },
    onResponse: (response) => {
        console.log(`打印响应: ${response}`);
    },
    onErrorResponse: (errorCode) => {
        console.log(`打印错误: ${errorCode}`);
    }
});
```


### 取消任务

```js
AvocadoSDK.getJobInfo(this.currentJobId, {
    onResponse: (response) => {
        console.log(`任务信息: ${response}`);
    },
    onErrorResponse: (errorCode) => {
        console.log(`获取任务信息错误: ${errorCode}`);
    }
});
```

### 获取任务信息

```js
AvocadoSDK.getJobInfo(this.currentJobId, {
    onResponse: (response) => {
        console.log(`任务信息: ${response}`);
    },
    onErrorResponse: (errorCode) => {
        console.log(`获取任务信息错误: ${errorCode}`);
    }
});
``` 

### 恢复任务

```js
AvocadoSDK.resume({
    onResponse: (response) => {
        console.log(`恢复任务成功`);
    },
    onErrorResponse: (errorCode) => {
        console.log(`恢复任务错误: ${errorCode}`);
    }
});
```

### 清理设备

```js
AvocadoSDK.clean({
    onResponse: (response) => {
        console.log(`清理设备成功`);
    },
    onErrorResponse: (errorCode) => {
        console.log(`清理设备错误: ${errorCode}`);
    }
});
```



