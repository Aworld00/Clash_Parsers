parsers: # array
  # - reg: ^.*$ 匹配所有订阅，或  - url: https://example.com/profile.yaml 指定订阅
  #- reg: ^.*$
  - url: 
    # 删除服务商提供的策略组和规则
    code: |
      module.exports.parse = (raw, { yaml }) => {
        const rawObj = yaml.parse(raw)
        const groups = []
        const rules = []
        return yaml.stringify({ ...rawObj, 'proxy-groups': groups, rules })
      }
    # 建立自己的配置
    yaml:
      prepend-proxy-groups: # 建立策略组
        - name: ✈️ 开启代理
          type: select
          proxies:
            - 🀄 关闭代理
            - 🇭🇰 香港标准 IEPL 中继 1
            - 🇭🇰 香港标准 IEPL 中继 2
        - name: 🆎 广告拦截
          type: select
          proxies:
            - REJECT
            - 🀄 关闭代理
            - ✈️ 开启代理
        - name: 🀄 关闭代理
          type: select
          proxies:
            - DIRECT
            - ⚡ 新加坡节点
        - name: 🏊 B站番剧出差
          type: select
          proxies:
            - 🀄 关闭代理
            - 🇭🇰 香港高级 IEPL 中继 1
            - 🇨🇳 台湾高级 IEPL 中继 1
        - name: 🎲 Steam平台
          type: select
          proxies:
            - 🀄 关闭代理
            - 🇭🇰 香港高级 IEPL 中继 1
        - name: 🎮 Epic游戏
          type: select
          proxies:
            - ✈️ 开启代理
            - 🀄 关闭代理
        - name: 🌍 国外网站
          type: select
          proxies:
            - ✈️ 开启代理
            - 🀄 关闭代理
        - name: 🐟 遗漏网站
          type: select
          proxies:
            - ✈️ 开启代理
            - 🀄 关闭代理
            - 🇭🇰 香港标准 IEPL 中继 1
        - name: ⚡ 新加坡节点
          type: url-test
          proxies:
            - 🇸🇬 新加坡标准 IEPL 中继 1
            - 🇸🇬 新加坡标准 IEPL 中继 2
            - 🇸🇬 新加坡标准 IEPL 中继 3
          url: http://www.apple.com/library/test/success.html
          interval: "3600"

        # 策略组示例
        # - name: 分组名
        # type: select       # 手动选点
        # url-test     # 自动选择延迟最低的节点
        # fallback     # 节点故障时自动切换下一个
        # laod-balance # 均衡使用分组内的节点
      # url: http://www.apple.com/library/test/success.html # 测试地址 非select类型分组必要
      # interval: 300 # 自动测试间隔时间，单位秒 非select类型分组必要
      # tolerance: 50 # 允许的偏差，节点之间延迟差小于该值不切换 非必要
      # proxies:
      # - 节点名称或其他分组套娃
      commands:
        - proxy-groups.🌍 国外网站.proxies.0+[]proxyNames # 向指定策略组添加订阅中的节点名，可使用正则过滤
        # - proxy-groups.✈️ 开启代理.proxies.0+DIRECT # 向指定分组第一个位置添加一个 DIRECT 节点名
        - proxy-groups.✈️ 开启代理.proxies.2+⚡ 新加坡节点
          # 一些可能用到的正则过滤节点示例，使分组更细致
          # []proxyNames|a                         # 包含a
          # []proxyNames|^(.*)(a|b)+(.*)$          # 包含a或b
          # []proxyNames|^(?=.*a)(?=.*b).*$        # 包含a和b
          # []proxyNames|^((?!b).)*a((?!b).)*$     # 包含a且不包含b
          # []proxyNames|^((?!b|c).)*a((?!b|c).)*$ # 包含a且不包含b或c
          # 更多正则教程，请看这里：https://deerchao.cn/tutorials/regex/regex.htm#top

          # 为各个策略组添加一个DIRECT，避免机场节点无法匹配上面的正则筛选而导致配置出错。应该有其他办法避免，但是我不会。
          # - proxy-groups.🇭🇰香港.proxies.0+REJECT

      # 添加规则
      prepend-rules: # 规则由上往下遍历，如上面规则已经命中，则不再往下处理
        #广告
        - RULE-SET,blackmatrix7_Reject,🆎 广告拦截
        - RULE-SET,blackmatrix7_Download,🀄 关闭代理,no-resolve
        #其他
        - DOMAIN-KEYWORD,epicgames.com,🎮 Epic游戏
        - DOMAIN-KEYWORD,ubisoft.com,🎲 Steam平台
        - DOMAIN-KEYWORD,ubi.com,🎲 Steam平台
        - DOMAIN-KEYWORD,quixel.com,🎮 Epic游戏
        #视频
        - RULE-SET,BilibiliHMT,🏊 B站番剧出差
        - RULE-SET,Bilibili,🀄 关闭代理,no-resolve
        #国内
        - RULE-SET,blackmatrix7_ChinaIPs_Classical,🀄 关闭代理,no-resolve
        - RULE-SET,blackmatrix7_China_Classical,🀄 关闭代理,no-resolve
        - RULE-SET,Tencent,🀄 关闭代理,no-resolve
        - RULE-SET,Design,🀄 关闭代理,no-resolve
        #软件
        - RULE-SET,Microsoft,🌍 国外网站
        - RULE-SET,OneDrive,🌍 国外网站
        - RULE-SET,Discord,🌍 国外网站
        - RULE-SET,blackmatrix7_Telegram,🌍 国外网站
        #游戏
        - RULE-SET,Steam,🎲 Steam平台
        - RULE-SET,blackmatrix7_Epic,🎮 Epic游戏
        - RULE-SET,Origin,🎲 Steam平台
        - RULE-SET,SteamCN,🀄 关闭代理,no-resolve
        #开启代理
        - RULE-SET,Github,✈️ 开启代理
        - RULE-SET,blackmatrix7_PROXY_Classical,✈️ 开启代理
        - RULE-SET,Small Mix,✈️ 开启代理
        #局域网
        - GEOIP,CN,🀄 关闭代理,no-resolve
        #其他
        - MATCH,🐟 遗漏网站 #⭐️⭐️规则之外的，在这里来修改是白名单模式还是黑名单模式，具体在“👆SELECT”选择代理或直连。


      # 添加规则集
        # DOMAIN-SUFFIX：域名后缀匹配
        # DOMAIN：域名匹配
        # DOMAIN-KEYWORD：域名关键字匹配
        # IP-CIDR：IP段匹配
        # SRC-IP-CIDR：源IP段匹配
        # GEOIP：GEOIP数据库（国家代码）匹配
        # DST-PORT：目标端口匹配
        # SRC-PORT：源端口匹配
        # PROCESS-NAME：源进程名匹配
        # RULE-SET：Rule Provider规则匹配
        # MATCH：全匹配


      # 添加规则集
      # name: # Provider 名称
      #   type: http # http 或 file
      #   behavior: classical # 或 ipcidr、domain
      #   path: # 文件路径
      #   url: # 只有当类型为 HTTP 时才可用，您不需要在本地空间中创建新文件。
      #   interval: # 自动更新间隔，仅在类型为 HTTP 时可用
      
      mix-rule-providers:
      # > 🆎 广告拦截
        blackmatrix7_Reject:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/AdvertisingLite/AdvertisingLite_Classical.yaml
          path: ./Rules/Reject
          interval: 86400
      # > 软件优化
        blackmatrix7_Download:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Download/Download.yaml
          path: ./Rules/Download
          interval: 86400
      # > 游戏平台
        blackmatrix7_Epic:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Epic/Epic.yaml
          path: ./Rules/Epic
          interval: 86400
        Steam:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/ACL4SSR/ACL4SSR@refs/heads/master/Clash/Providers/Ruleset/Steam.yaml
          path: ./Rules/Steam
          interval: 86400
        SteamCN:
          type: http
          behavior: classical
          url: https://fastly.jsdelivr.net/gh/Aworld00/ACL4SSR@master/Clash/Providers/Ruleset/SteamCN.yaml
          path: ./Rules/SteamCN
          interval: 86400
        Origin:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/ACL4SSR/ACL4SSR@refs/heads/master/Clash/Providers/Ruleset/Origin.yaml
          path: ./Rules/Origin
          interval: 86400
      # > 开启代理
        blackmatrix7_PROXY_Classical:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/Proxy/Proxy_Classical.yaml
          path: ./Rules/blackmatrix7_PROXY_Classical
          interval: 86400
      # > 海外媒体
      # > 海外小平台
        Small Mix:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/Aworld00/clash-a@refs/heads/main/GOOGLE_RULE-SET/%F0%9F%A7%A9Small%20Mix.yaml
          path: ./Rules/Small Mix
          interval: 86400
      # > 国内视频
        BilibiliHMT:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/ACL4SSR/ACL4SSR@refs/heads/master/Clash/Providers/Ruleset/BilibiliHMT.yaml
          path: ./Rules/BilibiliHMT.yaml
          interval: 86400
        Bilibili:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/ACL4SSR/ACL4SSR@refs/heads/master/Clash/Providers/Ruleset/Bilibili.yaml
          path: ./Rules/Bilibili
          interval: 86400
      # > 严选软件
        Discord:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/ACL4SSR/ACL4SSR@refs/heads/master/Clash/Providers/Ruleset/Discord.yaml
          path: ./Rules/Discord
          interval: 86400
        Tencent:
          type: http
          behavior: classical
          url: https://fastly.jsdelivr.net/gh/Aworld00/ACL4SSR@master/Clash/Providers/Ruleset/Tencent.yaml
          path: ./Rules/Tencent
          interval: 86400
        blackmatrix7_Telegram:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@release/rule/Clash/Telegram/Telegram.yaml
          path: ./Rules/Telegram
          interval: 86400
      # > 国内设计
        Design:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/Aworld00/clash-a@jojo/GOOGLE_RULE-SET/Design.yaml
          path: ./Rules/Design
          interval: 86400
      # > 知名品牌
        Github:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/GitHub/GitHub.yaml
          path: ./Rules/Github
          interval: 86400
        Microsoft:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/Aworld00/ACL4SSR@master/Clash/Providers/Ruleset/Microsoft.yaml
          path: ./Rules/Microsoft
          interval: 86400
        OneDrive:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/Aworld00/ACL4SSR@master/Clash/Providers/Ruleset/OneDrive.yaml
          path: ./Rules/OneDrive
          interval: 86400
      # > 国内直连
        blackmatrix7_China_Classical:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/China/China_Classical.yaml
          path: ./Rules/ChinaDomain
          interval: 86400
        blackmatrix7_ChinaIPs_Classical:
          type: http
          behavior: classical
          url: https://cdn.jsdelivr.net/gh/blackmatrix7/ios_rule_script@master/rule/Clash/ChinaIPs/ChinaIPs_Classical.yaml
          path: ./Rules/ChinaIPs_Classical
          interval: 86400
