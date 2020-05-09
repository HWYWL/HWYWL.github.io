---
title: PostgreSQL中jsonb数据格式操作
date: 2020-02-11 10:15:56
tags: 
 - PostgreSQL
categories: 
 - 数据库
keywords: "PostgreSQL"
description: PostgreSQL中jsonb数据格式操作。
---


## 建表SQL
```
CREATE TABLE "public"."contents_review_log" (
  "id" varchar(128) COLLATE "pg_catalog"."default" NOT NULL,
  "app_id" int4 NOT NULL,
  "open_id" varchar(128) COLLATE "pg_catalog"."default" NOT NULL,
  "platform_id" int4,
  "send_role_id" varchar(100) COLLATE "pg_catalog"."default" NOT NULL,
  "receive_role_id" varchar(100) COLLATE "pg_catalog"."default",
  "send_server_id" int4 DEFAULT 1,
  "receive_server_id" int4,
  "send_role_name" varchar(255) COLLATE "pg_catalog"."default",
  "receive_role_name" varchar(255) COLLATE "pg_catalog"."default",
  "send_vip_level" int4 DEFAULT 0,
  "receive_vip_level" int4 DEFAULT 0,
  "send_role_level" int4,
  "receive_role_level" int4,
  "chat_type" varchar(25) COLLATE "pg_catalog"."default",
  "suggestion" varchar(25) COLLATE "pg_catalog"."default",
  "detail" jsonb DEFAULT '{}'::jsonb,
  "chat_content" varchar(2400) COLLATE "pg_catalog"."default" NOT NULL,
  "event_time" timestamp(6) NOT NULL DEFAULT (now() + '08:00:00'::interval),
  "creation_time" timestamp(6) DEFAULT (now() + '08:00:00'::interval),
  "is_review" int2 NOT NULL DEFAULT 0,
  "generate_id" varchar(128) COLLATE "pg_catalog"."default",
  CONSTRAINT "review_contents_violation_pkey" PRIMARY KEY ("id")
)
;

ALTER TABLE "public"."contents_review_log" 
  OWNER TO "ikylin_nx";

CREATE INDEX "contents_review_log_index_chat_tpye" ON "public"."contents_review_log" USING btree (
  "chat_type" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
);

CREATE INDEX "contents_review_log_index_event_time" ON "public"."contents_review_log" USING btree (
  "event_time" "pg_catalog"."timestamp_ops" ASC NULLS LAST
);

CREATE INDEX "contents_review_log_index_generate_id" ON "public"."contents_review_log" USING btree (
  "generate_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
);

CREATE UNIQUE INDEX "contents_review_log_index_id" ON "public"."contents_review_log" USING btree (
  "id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
);

CREATE INDEX "contents_review_log_index_role" ON "public"."contents_review_log" USING btree (
  "send_role_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST,
  "receive_role_id" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
);

CREATE INDEX "contents_review_log_index_vip_level" ON "public"."contents_review_log" USING btree (
  "send_vip_level" "pg_catalog"."int4_ops" ASC NULLS LAST,
  "receive_vip_level" "pg_catalog"."int4_ops" ASC NULLS LAST
);

CREATE TRIGGER "auto_del_chat" AFTER INSERT ON "public"."contents_review_log"
FOR EACH ROW
EXECUTE PROCEDURE "public"."auto_del_chat"();

COMMENT ON COLUMN "public"."contents_review_log"."id" IS '唯一使用appid+违规角色id+服务器id+事件时间戳组合';

COMMENT ON COLUMN "public"."contents_review_log"."app_id" IS '游戏id';

COMMENT ON COLUMN "public"."contents_review_log"."open_id" IS '违规用户id';

COMMENT ON COLUMN "public"."contents_review_log"."platform_id" IS '平台id';

COMMENT ON COLUMN "public"."contents_review_log"."send_role_id" IS '违规角色id';

COMMENT ON COLUMN "public"."contents_review_log"."receive_role_id" IS '收信角色id';

COMMENT ON COLUMN "public"."contents_review_log"."send_server_id" IS '发送者所在服务器id';

COMMENT ON COLUMN "public"."contents_review_log"."receive_server_id" IS '接收者所在服务器id';

COMMENT ON COLUMN "public"."contents_review_log"."send_role_name" IS '发信者角色名称';

COMMENT ON COLUMN "public"."contents_review_log"."receive_role_name" IS '接收者角色名称';

COMMENT ON COLUMN "public"."contents_review_log"."send_vip_level" IS '违规角色vip等级';

COMMENT ON COLUMN "public"."contents_review_log"."receive_vip_level" IS '接收角色vip等级';

COMMENT ON COLUMN "public"."contents_review_log"."send_role_level" IS '发送者用户等级';

COMMENT ON COLUMN "public"."contents_review_log"."receive_role_level" IS '接收者用户等级';

COMMENT ON COLUMN "public"."contents_review_log"."chat_type" IS '频道';

COMMENT ON COLUMN "public"."contents_review_log"."suggestion" IS '检测结果可分为block、review';

COMMENT ON COLUMN "public"."contents_review_log"."detail" IS '语义检查结果';

COMMENT ON COLUMN "public"."contents_review_log"."chat_content" IS '审核文本';

COMMENT ON COLUMN "public"."contents_review_log"."event_time" IS '事件生成时间戳';

COMMENT ON COLUMN "public"."contents_review_log"."creation_time" IS '数据入库时间';

COMMENT ON COLUMN "public"."contents_review_log"."is_review" IS '后台是否已经处理';

COMMENT ON COLUMN "public"."contents_review_log"."generate_id" IS 'appid+违规角色id+服务器id+事件时间戳组合';

COMMENT ON TABLE "public"."contents_review_log" IS '内容审查表';
```

## JsonbTypeHandler解析
我们可以看到有一个叫detail的字段是使用了jsonb的格式存储数据，所以我们在JavaBean中可以这样建立一个模型。



```java
package com.jinkejoy.chat.aliyunchat.handler;

import cn.hutool.json.JSONUtil;
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedTypes;
import org.postgresql.util.PGobject;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * 自定义PSQL jsonb数据格式类型处理器
 *
 * @author huangwenyi
 * @date 2020-1-11
 */
@MappedTypes({Object.class})
public class JsonbTypeHandler extends BaseTypeHandler<Object> {

    private static final PGobject jsonObject = new PGobject();

    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Object o, JdbcType jdbcType) throws SQLException {
        jsonObject.setType("jsonb");
        jsonObject.setValue(JSONUtil.toJsonStr(o));
        preparedStatement.setObject(i, jsonObject);

    }

    @Override
    public Object getNullableResult(ResultSet resultSet, String s) throws SQLException {
        if (null != resultSet.getString(s)) {
            return JSONUtil.toBean(resultSet.getString(s), Object.class);
        }
        return null;

    }

    @Override
    public Object getNullableResult(ResultSet resultSet, int i) throws SQLException {
        if (null != resultSet.getString(i)) {
            return JSONUtil.toBean(resultSet.getString(i), Object.class);
        }
        return null;

    }

    @Override
    public Object getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        if (null != callableStatement.getString(i)) {
            return JSONUtil.toBean(callableStatement.getString(i), Object.class);
        }
        return null;
    }
}
```

## 使用
在model使用中使用mybatis plus中的 @TableField注解自定义json解析
```
package com.jinkejoy.chat.aliyunchat.model;

import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.jinkejoy.chat.aliyunchat.handler.JsonbTypeHandler;
import lombok.Data;

import java.io.Serializable;
import java.sql.Timestamp;
import java.util.Date;

/**
 * 数据库bean
 * @author huangwenyi
 */
@Data
public class ContentsReviewLog implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 唯一使用appid+违规角色id+服务器id+事件时间+随机数戳组合
     */
    @TableId
    private String id;

    /**
     * 游戏id
     */
    public Integer app_id;

    /**
     * 违规用户id
     */
    private String open_id;

    /**
     * 平台id
     */
    private Integer platform_id;

    /**
     * 违规角色id
     */
    private String send_role_id;

    /**
     * 收信角色id
     */
    private String receive_role_id;

    /**
     * 发送者所在服务器id
     */
    private Integer send_server_id;

    /**
     * 接收者所在服务器id
     */
    private Integer receive_server_id;

    /**
     * 发信者角色名称
     */
    private String send_role_name;

    /**
     * 接收者角色名称
     */
    private String receive_role_name;

    /**
     * 违规角色vip等级
     */
    private Integer send_vip_level;

    /**
     * 接收角色vip等级
     */
    private Integer receive_vip_level;

    /**
     * 发送者用户等级
     */
    private Integer send_role_level;

    /**
     * 接收者用户等级
     */
    private Integer receive_role_level;

    /**
     * 频道
     */
    private String chat_type;

    /**
     * 检测结果可分为block、review
     */
    private String suggestion;

    /**
     * 语义检查结果
     */
    @TableField(typeHandler = JsonbTypeHandler.class)
    private Detail detail;

    /**
     * 审核文本
     */
    private String chat_content;

    /**
     * 事件生成时间戳
     */
    private Timestamp event_time;

    /**
     * 数据入库时间
     */
    private Date creation_time = new Date();

    /**
     * 后台是否已经处理
     */
    private Integer is_review = 0;

    /**
     * appid+违规角色id+服务器id+事件时间
     */
    private String generate_id;

    public ContentsReviewLog() {
    }

}

```

## 结束
顺便说一句我在 **JsonbTypeHandler**类中使用的json工具是一个叫**hutool**的开源工具，你也可以使用**fastjson**代替也没问题。
使用上面这种方式可以像正常curd一样操作数据库，而不需要使用手写SQL的方式，简单快捷。