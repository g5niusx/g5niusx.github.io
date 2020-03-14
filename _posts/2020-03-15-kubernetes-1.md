---
layout: post
title: '使用 code generator 生成 kubernetes 的 crd 代码'
tags: [code]
---

## 创建一个 go mod 工程
初始的目录如下,**其中 hack 目录复制自k8s的官方[demo](https://github.com/kubernetes/sample-controller/tree/master/hack)**
创建的工程需要放在 `$GOPATH/src` 下面，否则生成的代码会在当前用户的根目录下
```text
.
├── go.mod
├── go.sum
├── hack
│   ├── boilerplate.go.txt
│   ├── tools.go
│   ├── update-codegen.sh
│   └── verify-codegen.sh
├── main.go
└── pkg
```

- go.mod

```text
module g5niusx.com/crd-demo

go 1.13

require (
	k8s.io/apimachinery v0.0.0-20200307122051-2b7fa1cb5395
	k8s.io/client-go v0.0.0-20200307122516-5194bac86967 // indirect
	k8s.io/code-generator v0.0.0-20200306081859-6a048a382944
)

replace (
	golang.org/x/sys => golang.org/x/sys v0.0.0-20190813064441-fde4db37ae7a // pinned to release-branch.go1.13
	golang.org/x/tools => golang.org/x/tools v0.0.0-20190821162956-65e3620a7ae7 // pinned to release-branch.go1.13
	k8s.io/api => k8s.io/api v0.0.0-20200307122242-510bcd53e1cf
	k8s.io/apimachinery => k8s.io/apimachinery v0.0.0-20200307122051-2b7fa1cb5395
	k8s.io/client-go => k8s.io/client-go v0.0.0-20200307122516-5194bac86967
	k8s.io/code-generator => k8s.io/code-generator v0.0.0-20200306081859-6a048a382944
)

```

## 创建 crd 资源需要的 types 定义、以及必须的 doc.go 、register.go 文件

```text
.
├── go.mod
├── go.sum
├── hack
│   ├── boilerplate.go.txt
│   ├── tools.go
│   ├── update-codegen.sh
│   └── verify-codegen.sh
├── main.go
└── pkg
    └── apis
        └── g5niusx
            ├── register.go
            └── v1
                ├── doc.go
                ├── register.go
                └── types.go
```

- v1/types.go 声明了 crd 的对象定义

```text
package v1

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// +genclient
// +genclient:noStatus
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
type MyCrd struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`
	Spec              MyCrdSpec `json:"spec"`
}

type MyCrdSpec struct {
	Name string `json:"name"`
}

// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
type MyCrdList struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ListMeta `json:"metadata"`
	Items           []MyCrd `json:"items"`
}
```

- g5niusx/register.go 声明了 group 和 version 的常量
```text
package g5niusx


// GroupName is the group name used in this package
const (
	GroupName = "g5niusx.com"
	Version = "v1"
)

```

- v1/doc.go 主要是通过 `+k8s:deepcopy-gen=package` 和 `+groupName=g5niusx.com` 对生成的代码表明了
深拷贝和生成的 group 名称
```text
// +k8s:deepcopy-gen=package
// +groupName=g5niusx.com
package v1
```

- v1/register.go 是将我们自定义的 `MyCrd`、`MyCrdList` 注册给 kubernetes

由于 `AddKnownTypes` 对于入参有要求，会有编译报错，等我们使用 `code-generator` 生成之后会自动实现 deepcopy 就不会有报错
```text
package v1

import (
	"g5niusx.com/crd-demo/pkg/apis/g5niusx"
	v1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/runtime/schema"
)

var (
	SchemeGroupVersion = schema.GroupVersion{Group: g5niusx.GroupName, Version: g5niusx.Version}
	// SchemeBuilder initializes a scheme builder
	SchemeBuilder = runtime.NewSchemeBuilder(addKnownTypes)
	// AddToScheme is a global function that registers this API group & version to a scheme
	AddToScheme = SchemeBuilder.AddToScheme
)

func Kind(kind string) schema.GroupKind {
	return SchemeGroupVersion.WithKind(kind).GroupKind()
}

func Resource(resource string) schema.GroupResource {
	return SchemeGroupVersion.WithResource(resource).GroupResource()
}

func addKnownTypes(scheme *runtime.Scheme) error {
	scheme.AddKnownTypes(SchemeGroupVersion, &MyCrd{}, &MyCrdList{})
	v1.AddToGroupVersion(scheme, SchemeGroupVersion)
	return nil
}
```

## 运行 code-generator

- 因为我们使用了 mod，所以在工程目录下先执行 `go mod vendor`，生成对应的vendor目录
```text
.
├── go.mod
├── go.sum
├── hack
├── main.go
├── pkg
└── vendor // 通过 go mod vendor 生成
```
- 修改 `hack/update-codegen.sh` 文件，主要是修改生成代码的位置和版本

```text
#!/usr/bin/env bash

# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set -o errexit
set -o nounset
set -o pipefail

SCRIPT_ROOT=$(dirname "${BASH_SOURCE[0]}")/..
CODEGEN_PKG=${CODEGEN_PKG:-$(cd "${SCRIPT_ROOT}"; ls -d -1 ./vendor/k8s.io/code-generator 2>/dev/null || echo ../code-generator)}
echo "脚本目录:$SCRIPT_ROOT"
echo "path :$CODEGEN_PKG"
# generate the code with:
# --output-base    because this script should also be able to run inside the vendor dir of
#                  k8s.io/kubernetes. The output-base is needed for the generators to output into the vendor dir
#                  instead of the $GOPATH directly. For normal projects this can be dropped.
bash "${CODEGEN_PKG}"/generate-groups.sh "deepcopy,client,informer,lister" \
  g5niusx.com/crd-demo/pkg/generated g5niusx.com/crd-demo/pkg/apis \
  g5niusx:v1 \
  --output-base "$(dirname "${BASH_SOURCE[0]}")/../../.." \
  --go-header-file "${SCRIPT_ROOT}"/hack/boilerplate.go.txt

# To use your own boilerplate text append:
#   --go-header-file "${SCRIPT_ROOT}"/hack/custom-boilerplate.go.txt
```

- 在工程的根目录下运行 `./hack/update-codegen.sh`

**一定要在工程的根目录下执行，否则会提示没有文件或者目录**

```text
   ./hack/update-codegen.sh
脚本目录:./hack/..
path :./vendor/k8s.io/code-generator
Generating deepcopy funcs
Generating clientset for g5niusx:v1 at g5niusx.com/crd-demo/pkg/generated/clientset
Generating listers for g5niusx:v1 at g5niusx.com/crd-demo/pkg/generated/listers
Generating informers for g5niusx:v1 at g5niusx.com/crd-demo/pkg/generated/informers
```
- 执行成功之后，就会在工程的 `pkg/generated`目录下生成对应的代码

```text
.
├── apis
│   └── g5niusx
│       ├── register.go
│       └── v1
│           ├── doc.go
│           ├── register.go
│           ├── types.go
│           └── zz_generated.deepcopy.go
└── generated
    ├── clientset
    │   └── versioned
    │       ├── clientset.go
    │       ├── doc.go
    │       ├── fake
    │       │   ├── clientset_generated.go
    │       │   ├── doc.go
    │       │   └── register.go
    │       ├── scheme
    │       │   ├── doc.go
    │       │   └── register.go
    │       └── typed
    │           └── g5niusx
    ├── informers
    │   └── externalversions
    │       ├── factory.go
    │       ├── g5niusx
    │       │   ├── interface.go
    │       │   └── v1
    │       ├── generic.go
    │       └── internalinterfaces
    │           └── factory_interfaces.go
    └── listers
        └── g5niusx
            └── v1
                ├── expansion_generated.go
                └── mycrd.go
```



