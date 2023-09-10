# 智能合约漏洞案例，EDE Finance 攻击事件分析

# 件背景

[Safful](https://safful.com/)区块链安全情报平台监控到消息，北京时间 2023 年 5 月 30 日，Arbitrum 链上 EDE Finance 项目受到黑客攻击，攻击者获利 597,694 USDC 及 86,222 USDT，攻击者地址为 0x80826e9801420e19a948b8ef477fd20f754932dc。**攻击者完成攻击后，在链上留言声称为“白帽行为”，目前攻击者已返还 EDE FinanceEDE Finance 项目 333,948 USDC 及 86,222 USDT，留下部分 USDC 未转移。**[Safful](https://safful.com/)安全团队及时对此安全事件进行分析。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317812835-b1a9e285-ffcd-4e1e-8d3d-aaf50c8066be.png#averageHue=%23fefefd&clientId=u033c60fa-98a0-4&from=paste&height=214&id=u4a0f5bc3&originHeight=519&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=95761&status=done&style=none&taskId=u110bbcc1-351e-4c9f-9bdc-1464c16f5c9&title=&width=445)

# 攻击过程分析

- **攻击时间背景**

攻击者提前创建好攻击合约：
0x6dd3d2fb02b0d7da5dd30146305a14190e6fb892
创建攻击合约的交易 hash:
0xd7bfe2ed548c40d3e9b301a83f16e9c4080e151470ebf4d03c83ff302b9ad99e
攻击合约是一个代理合约，其主要逻辑合约位于：
0x171c01883460b83144c2098101cd57273b72a054，未做合约验证
攻击者其中一组攻击交易：
0xc3677aec922f3c0641176277d9923bf03ff400e9b79013ba86ac8ceb878fcd86
0x6e48dbe65c9997d774d75bc01468a354c3660a44eef59a4383639933a50fc814
EDE 项目的价格预言机合约 **VaultPriceFeedV21Fast ：**
0x046600975bed388d368f843a67e41545e27a2591
EDE 项目的**Router**合约：
0x2c7077cf9bd07c3bc45b4e5b8c27f8b95c6550b3
EDE 项目的 **RouterSign** 合约：
0xd067e4b0144841bc79153874d385671ea4c4e4df ，此合约地址拥有 Updater 管理员权限，该权限可以进行加密资产的价格设置，如下图所示。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317822798-9e52beb2-8675-4d95-af91-2f0000042725.png#averageHue=%23171819&clientId=u033c60fa-98a0-4&from=paste&height=100&id=u9a69fd36&originHeight=243&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=29269&status=done&style=none&taskId=u3677e35a-aa7d-4c9b-840b-9c3fe6f5fac&title=&width=444)

- **攻击步骤**

  1.攻击者部署攻击合约
  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317828397-b41dffb2-07e4-4615-ae70-5882c7aef7a4.png#averageHue=%23fefdfd&clientId=u033c60fa-98a0-4&from=paste&height=152&id=u64ccb662&originHeight=304&originWidth=949&originalType=binary&ratio=2&rotation=0&showTitle=false&size=35105&status=done&style=none&taskId=uac2bb37c-1155-464e-9c8e-da5efd66002&title=&width=475) 2.攻击者通过调用攻击合约，间接调用了 RouterSign 合约中的添加仓位操作
  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317834663-71e4b42a-79c8-47b9-b0ce-a6ef3a09303c.png#averageHue=%23fefdfd&clientId=u033c60fa-98a0-4&from=paste&height=201&id=u72cf0917&originHeight=402&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=81654&status=done&style=none&taskId=u3c216d03-36bb-4cc2-9fec-242239a9510&title=&width=540)
  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317840736-0c821f9f-c32e-4985-a310-ad181ea61031.png#averageHue=%2324272f&clientId=u033c60fa-98a0-4&from=paste&height=156&id=ua80b74f2&originHeight=312&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=281959&status=done&style=none&taskId=u870118fa-4ff8-4796-b7d3-ee3d893fdc2&title=&width=540) 3.随后攻击者立即又利用攻击合约调用了 Router 合约的减仓操作
  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317846420-91516e20-c042-42a3-b582-840a8339f7ff.png#averageHue=%23fefdfd&clientId=u033c60fa-98a0-4&from=paste&height=163&id=uc35cf668&originHeight=326&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=82550&status=done&style=none&taskId=u590d1801-c0d1-47f9-b573-65b1fdeb5d0&title=&width=540)
  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317851826-0674babc-ce4c-4e7c-a4e8-4daa567c10f2.png#averageHue=%2322252c&clientId=u033c60fa-98a0-4&from=paste&height=194&id=u752e274a&originHeight=388&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=322266&status=done&style=none&taskId=u86f774b5-897c-4197-a5a9-d8e73091268&title=&width=540) 4.后续，攻击者通过重复上述攻击步骤，通过多次加仓减仓，进行大额获利
  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317857607-a83f58a9-ae9d-4979-8a0c-b056be368236.png#averageHue=%23decbb2&clientId=u033c60fa-98a0-4&from=paste&height=292&id=u7b12407e&originHeight=584&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=225130&status=done&style=none&taskId=u9fdd7f73-3a26-439d-b251-da8c95144c3&title=&width=540)

# 核心漏洞

根据对攻击交易分析，发现攻击者通过加仓与减仓直接使用的价格差从而获取巨额资金，具体如下分析过程。
**加仓时使用价格：**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317864331-76e8fa2f-2cec-4601-9fb5-1ac78d271d88.png#averageHue=%23272a31&clientId=u033c60fa-98a0-4&from=paste&height=34&id=u9889c569&originHeight=82&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=85196&status=done&style=none&taskId=u62896439-a1a5-4fab-9800-5196d369aab&title=&width=447)
**减仓时使用价格：**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317868923-10f3fca0-0b5f-4c33-b763-425c9f7239fb.png#averageHue=%23262930&clientId=u033c60fa-98a0-4&from=paste&height=33&id=u6410ee2e&originHeight=66&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=61893&status=done&style=none&taskId=u453b7d21-0d0c-4c7f-bee9-04033763529&title=&width=540)
加仓和减仓的价格明显存在较大差距。
通过对攻击者交易的调用堆栈发现，其在加仓时，通过调用了 EDE 项目中价格预言机的更新价格功能 updateWithSig ，而预言机的喂价功能不仅限制了必须存在 Updater 特权角色，且必须使用 Updater 角色进行签名才可以正常调用。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317874769-93fae650-b3f4-4fac-a56f-1c6db68369dd.png#averageHue=%23252524&clientId=u033c60fa-98a0-4&from=paste&height=40&id=u353091d6&originHeight=98&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=42610&status=done&style=none&taskId=u395780cf-be9e-4f6c-a7ca-11144702891&title=&width=445)
但攻击者利用了 RouterSign 合约的加仓函数，间接调用 价格预言机中的 updateWithSig 特权函数，从而成功绕过了原本的 onlyUpdater 权限限制。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317881013-42a930f4-89c1-433e-ac90-6ceec1657b48.png#averageHue=%23232322&clientId=u033c60fa-98a0-4&from=paste&height=114&id=u1744037f&originHeight=227&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=90811&status=done&style=none&taskId=u9d0a4e35-d959-4b1a-b2c1-9795599d056&title=&width=540)
并且由于 EDE 价格预言机中喂价签名验证机制存在问题，虽然对签名时间进行了校验，但未对已经使用过的签名进行校验，导致攻击者可以重放管理员签名。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317886866-a2a89818-973e-4597-89f7-bab78ee51ba0.png#averageHue=%23222120&clientId=u033c60fa-98a0-4&from=paste&height=466&id=ude6c8151&originHeight=930&originWidth=651&originalType=binary&ratio=2&rotation=0&showTitle=false&size=89700&status=done&style=none&taskId=uac7c9991-e137-4e20-ab0a-c8a30a591df&title=&width=326)
如下图为攻击者间接调用预言机喂价功能传递的签名参数：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317892785-131e2c57-b543-49d8-9ab8-f5b527d59d7b.png#averageHue=%231e1e23&clientId=u033c60fa-98a0-4&from=paste&height=247&id=u581770c0&originHeight=493&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=107948&status=done&style=none&taskId=u62e15a22-c96b-40bf-9b54-e4a3f43332d&title=&width=540)
如下图为 EDE 项目管理员通过预言机合约直接调用喂价功能使用的参数：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317898849-17629ad3-ad9e-470c-be95-67bf8d0ed65b.png#averageHue=%231e1e23&clientId=u033c60fa-98a0-4&from=paste&height=243&id=u2272297a&originHeight=486&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=95494&status=done&style=none&taskId=uc40aa789-13bb-401c-8493-8a55b0e6939&title=&width=540)
所以，综上所述，攻击者使用管理员签名，并且绕过 onlyupdater 限制，成功重放并设置了一个较高的 Token 价格，并在随后立即使用新价格执行加仓操作，然后攻击者立即调用 Router 合约使用正常的价格进行减仓，利用较大的价格差从而获利。

# 资金来源及流向

通过分析发现，攻击地址初始手续费通过 0x483e657f53c7bc00c2d2a26988749dd8846229e3 地址转入 0.002ETH
0x483e657f53c7bc00c2d2a26988749dd8846229e3 地址的初始手续费通过 Kucoin 交易所提取
![image.png](https://cdn.nlark.com/yuque/0/2023/png/97322/1694317905892-9a769e9a-631f-4150-90f2-d64e3be66d3b.png#averageHue=%23fefefd&clientId=u033c60fa-98a0-4&from=paste&height=329&id=u5dfff660&originHeight=657&originWidth=1080&originalType=binary&ratio=2&rotation=0&showTitle=false&size=116985&status=done&style=none&taskId=ub7167bf3-59db-498c-8f2a-ca07544e964&title=&width=540)

# 总结及建议

- **分析总结**

本次攻击是由于项目新增合约时未进行安全审计，新增合约破坏原有合约权限认证机制，配合 Updater 签名重放问题，从而导致这两个利用点被组合利用。
**攻击利用点：**

- RouterSign 合约中对 updateWithSig 的 Updater 特权限制绕过
- Updater 特权签名重放

**安全建议**

1. 项目价格预言机喂价的签名验证添加唯一标识符，并限制每个签名只能使用一次；
2. 项目需要确认 RouterSign 合约中的功能函数是否能外部公开，且设置权限校验。
