+ [2020-01-17](#2020-01-17)
+ [2020-01-19](#2020-01-19)
+ [2020-01-20](#2020-01-20)

---
### 2020-01-17
+ 工作相关
    + 告警代码阅读
        - [ ] ~~10:40 —— 11:20~~ 
        - [x] 11:30 —— 12:00
        - [x] 16:40 —— 18:00
    + 告警相关文档整理
        - [ ] ~~17:10 —— 18:30~~
    + 文档整理+邮件发送
        - [x] 13:30 —— 14:30 邮件等着后续再发
        - [ ] ~~14:00 —— 16:00~~
+ 学习相关
    + SOA相关知识学习
        - [x] 09:30 —— 10:30 **还是没有特别看明白**
    + 分布式架构音频
        - [ ] ~~16:10 —— 17:10~~
+ 杂七杂八
    + 10:40 —— 11:20
        - 搞TODO LIST
    + 14:30 —— 16:40
        - am 告警 需求会
+ 小结
    + 做了哪些
        + [x] 09:30 —— 10:30 SOA相关知识学习
        + [x] 10:40 —— 11:20 TODO LIST编辑
        + [x] 11:30 —— 12:00 告警代码阅读
        + [x] 13:30 —— 14:30 文档整理邮件等着后续再发
        + [x] 14:30 —— 16:40 am 告警 需求会
        + [x] 16:40 —— 18:00 告警代码阅读
    + 没做哪些
        + 告警代码没按计划定量阅读
        + 分布式架构音频
        + 告警相关文档整理
    + 主要原因
        + 构建TODO LIST 结构浪费40分钟时间
        + 开会占用了2小时时间
    + 明日TODO 指导
        + 预留一部分时间用于响应突发事件（开会等）
        + SOA相关知识学习
---

### 2020-01-19

+ 工作相关
    - [x] 09:00 —— 09:30 早会 + TODO LIST 整理
    - [x] 09:30 —— 11:30 告警代码阅读
    - [x] 11:30 —— 12:00 告警文档整理
    - [x] 13:30 —— 17:00 告警代码阅读
    - [x] 17:00 —— 18:00 告警文档整理
+ 学习相关
    - [ ] SOA相关知识学习
    - [ ] 分布式架构限流算法相关
    - [x] COLLSHELL TCP/IP文章阅读 [TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)
    - [ ] COLLSHELL TCP/IP文章阅读 [TCP 的那些事儿（下）](https://coolshell.cn/articles/11609.html)
---

### 2020-01-20
+ 工作相关
    - [x] 09:00 —— 09:30 早会 + TODO LIST 整理
    - [x] 09:30 —— 11:30 告警代码阅读
    - [x] 11:30 —— 12:00 告警文档整理
    - [x] 13:30 —— 17:00 告警代码阅读
    - [x] 17:00 —— 18:00 告警文档整理
+ 学习相关
    - [ ] SOA相关知识学习
    - [ ] 分布式架构限流算法相关
    - [x] COLLSHELL TCP/IP文章阅读 [TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)
    - [ ] COLLSHELL TCP/IP文章阅读 [TCP 的那些事儿（下）](https://coolshell.cn/articles/11609.html)
```
@startuml
  start
    :agent_service;
    :post "/service";
    note right
      init alarm_service by agent post
    end note
    :DDI::Alarm.instance.start_monitor_thread;
    note right
      start monitor thread in post request
    end note
    fork
      :set @email_alarm_info and every alarm level last sent time;
        note right
          ====
               def reset_email_send_time(last_send_time = nil)
                    if @email_alarm_info.nil?
                        @email_alarm_info = Hash.new
                    end
                    if last_send_time.nil?
                        last_send_time = Time.parse("1937-09-01 00:00:00")
                    end
                    AlarmLevel.reverse.each{|name|
                        info = {
                            :frequency => @email_alarm.get_alarm_frequency(name).to_i,
                            :last_send_time => last_send_time,
                        }
                        @email_alarm_info[name] = info
                    }
                end
          
        end note
      :init global_alarm_msgs;
        note right
          create an object of AlarmInfo
          ====
            @global_alarm_msgs = AlarmInfo.new;
        end note
      while(@is_running)
        note right 
          @is_running default is true and also can receive signial to set it's value(true/false)
        end note 
      :sleep 60 sec;
      :do nothing if current HA role is backup and call next;
      :loop counter increase and call GC.start if counter%10 = 0;
      :the varable {alarm_cnt} is a counter which set thread goes down when it's value >= 3;
      :init some massage varables;
        note right
         * @curr_ws_msg = "" **WebService alarm message**
         * @curr_sound_msg = "" **sound alarm message**
         * @curr_ws_msg_zh = "" **WebService alarm message for zh**
        end note
      :init @global_dhcp_usage["shared_network_dhcp_usage"];
        note right 
          @global_dhcp_usage is an empty hash by default, get **shared_network_dhcp_usage** by http request
          ====
            def get_shared_network_dhcp_usage
              run_command("get", LOCAL_IP, SERVICE_PORT["cloudipam"], "shared-network-dhcp-usage", {})
            end
        end note
      :get all member info;
        note right
          get all members info by making a http request, and 
          ====
          def get_all_member_info
            @groups = nil
            params = {"current_user"=>"admin", "need_info" => "yes",
                      "need_warn" => "no",
                      "need_clear_dns_log_control_warns" => "yes"}
            response = run_command("get", LOCAL_IP, SERVICE_PORT["grid"], "groups", params)
            return if response.empty?
            #这里产生时间戳，下发到各节点，供节点ha或slave备份到master时校验数据一致性时使用
            @sync_check_time = Time.now.strftime('%F %T')
            all_member_request_success = true
            @groups = response["resources"]
            @groups.each do |group|
                if group["members"].nil?
                    next
                end
                for i in 0...group["members"].length
                    begin
                        member = group["members"][i]
                        next if !member["is_connect"]

                        server_is_running = member["is_connect"] ? DDI::HostNetworkHelper.host_accessible?(
                            member["ip"], 1, nil, true) : false
                        state = member["is_connect"] ? (server_is_running ? STATE_ONLINE : STATE_OFFLINE) : STATE_DISCONNECT
                        member["state"] = state
                        next if member["state"] != STATE_ONLINE

                        member["owner"] = "#{group['id']}.#{member['id']}"
                        group["members"][i].merge!(fetch_member_info(member))
                    rescue Exception => e
                        all_member_request_success = false
                        DDI::Log.error "#{e.to_s}\n#{$@[0..20]}"
                    end
                end
            end
            
            #when any member request fail, don't alarm discover 
            @groups.each do |group|
                if group["members"].nil?
                    next
                end
                for i in 0...group["members"].length
                    if group["members"][i]["discover_result"].nil? || 
                        group["members"][i]["discover_result"].empty?
                        next
                    end

                    if all_member_request_success == false
                        group["members"][i]["discover_result"] = []
                    else
                        @members_info.values.each do |info|
                            group["members"][i]["discover_result"] -= (info["nic_ips"] || [])
                        end
                     end
                end
            end
          end
        end note
      endwhile
    end fork
  stop
@enduml
```
---

### 2020-01-21

+ 工作相关
    - [x] 09:00 —— 09:30 早会 + TODO LIST 整理
    - [ ] 09:30 —— 11:30 告警代码阅读
    - [ ] 11:30 —— 12:00 告警文档整理
    - [ ] 13:30 —— 17:00 告警代码阅读
    - [ ] 17:00 —— 18:00 告警文档整理
+ 学习相关
    - [ ] SOA相关知识学习
    - [ ] 分布式架构限流算法相关
    - [x] COLLSHELL TCP/IP文章阅读 [TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)
    - [ ] COLLSHELL TCP/IP文章阅读 [TCP 的那些事儿（下）](https://coolshell.cn/articles/11609.html)