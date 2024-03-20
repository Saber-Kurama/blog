

## 安装和使用

https://docs.github.com/zh/copilot/quickstart
### 问题

1. 网络原因不稳定  @郭超 uniapp 
2.  服务器的成本费用
3. 


## 使用场景

1. 变量名和方法名
2. 代码生成注释
3.  检查代码， 查询解决方案
4.   

## 技巧


[# GitHub Copilot：02 必须知道的7个技巧帮助你代码自动补全](https://ducafecat.com/blog/7-must-know-tips-to-help-you-autocomplete-your-code-with-github-copilot-02)

一半代码完成后续 @朱勇补充文档





## 例子

1. 使用 copilot 的的

## 待解决

1. 一半代码完成后续 @
2. 如何喂文档
3.  可以抽象一些共用 ？ 提效/ 
4. 通文千义 和  百度 和 copolit  
5. Prompt 工程 提示






## 结论

1. 准确性更好
2. 速度慢


copolit > 通文千义 > 百度




AI 生产 代码 



一些demo


提问

``` ts
import { Button, Menu } from "antd";
import type { MenuProps } from "antd";
import { useRouter } from 'next/navigation'
import { CaretUpOutlined, ProductOutlined } from "@ant-design/icons";

type MenuItem = Required<MenuProps>["items"][number] & { isExternal?: boolean, href?: string, children?: MenuItem[] };

export const SideBarMenuData: MenuItem[] = [
  {
    key: "digitalWorkbench_fetch",
    // icon,
    // shi'fo
    // 是否是外部链接
    isExternal: true,
    // 外部链接地址
    href: "https://prod.digitforce.com/dap-hm/#/digitalWorkbench/fetch",
    label: "工作台",
    icon: <ProductOutlined />
  },
  {
    key: "2",
    label: "指标工厂",
    icon: <ProductOutlined />,
    children: [
      {
        // https://prod.digitforce.com/dap-hm/#/factory/index
        key: "factory_index",
        label: "指标管理",
        isExternal: true,
        href: "https://prod.digitforce.com/dap-hm/#/factory/index",
        icon: <ProductOutlined />,
      },
      {
        // https://prod.digitforce.com/dap-hm/#/factory/dimension
        key: "factory_dimension",
        label: "维度管理",
        isExternal: true,
        href: "https://prod.digitforce.com/dap-hm/#/factory/dimension",
        icon: <ProductOutlined />,
      },
      {
        key: "factory_model",
        label: "模型管理",
        // https://prod.digitforce.com/dap-hm/#/factory/model
        isExternal: true,
        href: "https://prod.digitforce.com/dap-hm/#/factory/model",
        icon: <ProductOutlined />,
      },
      {
        key: "factory_data_table-management",
        label: "数据管理",
        // https://prod.digitforce.com/dap-hm/#/factory/data/table-management
        isExternal: true,
        href: "https://prod.digitforce.com/dap-hm/#/factory/data/table-management",
        icon: <ProductOutlined />,
      },
    ],
  },
  {
    key: "3",
    label: "知识库配置",
    icon: <ProductOutlined />,
  },
];

把这段代码中[https://prod.digitforce.com](https://prod.digitforce.com/) 替换成 process.env.NEXT_PUBLIC_PLATFORM_API
```



通义灵码

``` ts

import { Button, Menu } from "antd";
import type { MenuProps } from "antd";
import { useRouter } from 'next/navigation'
import { CaretUpOutlined, ProductOutlined } from "@ant-design/icons";

type MenuItem = Required<MenuProps>["items"][number] & { isExternal?: boolean, href?: string, children?: MenuItem[] };

// 假设 process.env.NEXT_PUBLIC_PLATFORM_API 的值是 API 的基础 URL
const platformAPI = process.env.NEXT_PUBLIC_PLATFORM_API || '';

export const SideBarMenuData: MenuItem[] = [

  {
    key: "digitalWorkbench_fetch",
    isExternal: true,
    href: `${platformAPI}/dap-hm/#/digitalWorkbench/fetch`,
    label: "工作台",
    icon: <ProductOutlined />,
  },

  {
    key: "2",
    label: "指标工厂",
    icon: <ProductOutlined />,
    children: [
      {
        key: "factory_index",
        label: "指标管理",
        isExternal: true,
        href: `${platformAPI}/dap-hm/#/factory/index`,
        icon: <ProductOutlined />,
      },
      {
        key: "factory_dimension",
        label: "维度管理",
        isExternal: true,
        href: `${platformAPI}/dap-hm/#/factory/dimension`,
        icon: <ProductOutlined />,
      },
      {
        key: "factory_model",
        label: "模型管理",
        isExternal: true,
        href: `${platformAPI}/dap-hm/#/factory/model`,
        icon: <ProductOutlined />,
      },
      {
        key: "factory_data_table-management",
        label: "数据管理",
        isExternal: true,
        href: `${platformAPI}/dap-hm/#/factory/data/table-management`,
        icon: <ProductOutlined />,
      },
    ],
  },

  {
    key: "3",
    label: "知识库配置",
    icon: <ProductOutlined />,
  },
];

```


copolit

``` ts
import { Button, Menu } from "antd";
import type { MenuProps } from "antd";
import { useRouter } from 'next/navigation'
import { CaretUpOutlined, ProductOutlined } from "@ant-design/icons";

type MenuItem = Required<MenuProps>["items"][number] & { isExternal?: boolean, href?: string, children?: MenuItem[] };

export const SideBarMenuData: MenuItem[] = [
  // Your data...
].map(item => {
  if (item.href) {
    item.href = item.href.replace('https://prod.digitforce.com', process.env.NEXT_PUBLIC_PLATFORM_API);
  }

  if (item.children) {
    item.children = item.children.map(child => {
      if (child.href) {
        child.href = child.href.replace('https://prod.digitforce.com', process.env.NEXT_PUBLIC_PLATFORM_API);
      }
      return child;
    });
  }

  return item;
});
```

明确指出是 环境变量替换

``` ts
import { Button, Menu } from "antd";
import type { MenuProps } from "antd";
import { useRouter } from 'next/navigation'
import { CaretUpOutlined, ProductOutlined } from "@ant-design/icons";

type MenuItem = Required<MenuProps>["items"][number] & { isExternal?: boolean, href?: string, children?: MenuItem[] };

export const SideBarMenuData: MenuItem[] = [
  {
    key: "digitalWorkbench_fetch",
    isExternal: true,
    href: process.env.NEXT_PUBLIC_PLATFORM_API + "/dap-hm/#/digitalWorkbench/fetch",
    label: "工作台",
    icon: <ProductOutlined />
  },
  {
    key: "2",
    label: "指标工厂",
    icon: <ProductOutlined />,
    children: [
      {
        key: "factory_index",
        label: "指标管理",
        isExternal: true,
        href: process.env.NEXT_PUBLIC_PLATFORM_API + "/dap-hm/#/factory/index",
        icon: <ProductOutlined />,
      },
      {
        key: "factory_dimension",
        label: "维度管理",
        isExternal: true,
        href: process.env.NEXT_PUBLIC_PLATFORM_API + "/dap-hm/#/factory/dimension",
        icon: <ProductOutlined />,
      },
      {
        key: "factory_model",
        label: "模型管理",
        isExternal: true,
        href: process.env.NEXT_PUBLIC_PLATFORM_API + "/dap-hm/#/factory/model",
        icon: <ProductOutlined />,
      },
      {
        key: "factory_data_table-management",
        label: "数据管理",
        isExternal: true,
        href: process.env.NEXT_PUBLIC_PLATFORM_API + "/dap-hm/#/factory/data/table-management",
        icon: <ProductOutlined />,
      },
    ],
  },
  {
    key: "3",
    label: "知识库配置",
    icon: <ProductOutlined />,
  },
];
```