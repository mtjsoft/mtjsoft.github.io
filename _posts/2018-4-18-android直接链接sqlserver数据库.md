---
layout: post
title:  "android直接链接sqlserver数据库"
subtitle:  "android sqlserver 数据库"
date:  2018-04-18
author:  "Mtj"
tags:
     android
     sqlserver
     数据库
     
---

## 1、开发中遇到需要android端直接链接服务端sqlserver数据库，并向sqlserver数据库插入数据的场景。

### 2、链接到sqlserver数据库，需要用到 jtds 工具包，我使用的是 [jtds-1.3.1.jar](https://pan.baidu.com/s/1ihwXmVNG-YyJTKlTwrPz1w)。点击百度网盘下载，密码： fma7

### 3、下面就是直接链接sqlserver数据库的代码：

```
package com.mtjsoft.www.myapplication.data;

import android.util.Log;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

/**
 * @author mtj 2018-4-9 10:00:48
 *         Created by Administrator on 2018/4/9.
 *         android 链接sqlserver数据库
 */

public class MtjServerDatabaseTools {
    private static String user = "数据库用户名";
    private static String password = "数据库密码";
    private static String DatabaseName = "数据库名称";
    private static String IP = "数据库所在的IP地址";
    /**
     * 连接字符串
     */
    private static String connectDB = "jdbc:jtds:sqlserver://" + IP + ":1433/" + DatabaseName + ";useunicode=true;characterEncoding=UTF-8";

    private static Connection conn = null;
    private static Statement stmt = null;

    /**
     * 链接数据库
     *
     * @return
     */
    private static Connection getSQLConnection() {
        Connection con = null;
        try {
            //加载驱动换成这个
            Class.forName("net.sourceforge.jtds.jdbc.Driver");
            //连接数据库对象
            con = DriverManager.getConnection(connectDB, user,
                    password);
        } catch (Exception e) {
        }
        return con;
    }

    /**
     * 向服务器数据库插入数据
     * @tabName 要插入的表名
     * @tabTopName 要插入的字段名字符串，例如（name,password,age）
     * @values 与tabTopName 中 字段名一一对应的值。一次插入多跳数据，可以用逗号隔开。例如（"张三"，"zhangsan","24"），（"李四"，"lisi","26"）
     */
    public static int insertIntoData(String tabName, String tabTopName, String values) {
        int i = 0;
        try {
            if (conn == null) {
                conn = getSQLConnection();
                stmt = conn.createStatement();
            }
            if (conn == null || stmt == null) {
                return i;
            }
            String sql = "insert into " + tabName + tabTopName + " values " + values;
            i = stmt.executeUpdate(sql);
        } catch (SQLException e) {
            e.printStackTrace();
            Log.i("mtj", "同步数据库表【" + tabName + "】失败。");
        }
        return i;
    }

    /**
     * 向服务器数据库插入数据---并返回插入的主键id。
     * 注意，此方法只能返回最新插入的一条数据的id。所以一次插入多跳数据时，只返回最后一个id。
     */
    public static int insertIntoDataReturnId(String tabName, String tabTopName, String values, CallBackImp callBackImp) {
        int id = 0;
        try {
            if (conn == null) {
                conn = getSQLConnection();
                stmt = conn.createStatement();
            }
            if (conn == null || stmt == null) {
                return id;
            }
            String sql = "insert into " + tabName + tabTopName + " values " + values;
            int i = stmt.executeUpdate(sql);
            if (i > 0) {
                ResultSet resultSet = stmt.executeQuery("select SCOPE_IDENTITY() as id;");
                while (resultSet.next()) {
                    id = resultSet.getInt(1);
                }
            } else {
                Log.i("mtj", "同步数据库表【" + tabName + "】---->失败。");
            }
        } catch (SQLException e) {
            e.printStackTrace();
            Log.i("mtj", "同步数据库表【" + tabName + "】---->失败。");
        }
        return id;
    }

    /**
     * 关闭数据库链接
     */
    public static void closeConnect() {
        if (stmt != null) {
            try {
                stmt.close();
                stmt = null;
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn != null) {
            try {
                conn.close();
                conn = null;
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

```
### android端连接sqlserver数据库的方法就完成了，是不是很简单。

### 4、我们模仿一个场景演示一下。从本地数据库user表中取出所有数据，插入到sqlserver数据中。
* 从本地取出所有的user数据。转换成上面 insertIntoData（）方法中所需要传入的字符串。
*  假设本地user表和sqlserver中的user表的字段都是一致的。

```
            SQLiteDatabase database = 获取一个可读的就行，这里就不说;
            //数据库表的所有字段名称
            StringBuilder tabTopName = new StringBuilder();
            //存放数据库表的数据
            StringBuilder values = new StringBuilder();
            //按照表名user，查询出所有数据
            Cursor cursor = database.query("user", null, "",
                    null, null, null, null);
            if (cursor != null && cursor.getCount() > 0) {
                int column = cursor.getColumnCount();
                //拼接所有字段名，例如 （name,password,age）
                tabTopName.append("(");
                for (int k = 0; k < column; k++) {
                    String s = cursor.getColumnName(k);
                    //过滤掉本地数据库的id；因为sqlserver有自增的id
                    if (!"id".equals(s)) {
                        tabTopName.append(s);
                        if (k != column - 1) {
                            tabTopName.append(",");
                        } else {
                            tabTopName.append(")");
                        }
                    }
                }
                //拼接所有数据
                //例如（"张三"，"zhangsan","24"），（"李四"，"lisi","26"）
                while (cursor.moveToNext()) {
                    values.append("(");
                    for (int k = 0; k < column; k++) {
                        String s = cursor.getColumnName(k);
                        //过滤掉本地数据库的id；因为sqlserver有自增的id
                        if (!"id".equals(s)) {
                            String strValue = cursor.getString(k);
                            values.append("'");
                            values.append(strValue);
                            values.append("'");
                            if (k != column - 1) {
                                values.append(",");
                            } else {
                                values.append(")");
                            }
                        }
                    }
                    values.append(",");
                }
                values.deleteCharAt(values.length() - 1);
                cursor.close();
                database.close();
                //组合成sql语句的格式。向服务器数据库插入数据。
                int count = MtjServerDatabaseTools.insertIntoData("user", tabTopName.toString(), values.toString());
                if (count > 0) {
                    //同步数据库表成功
                } else {
                    //同步数据库表失败
                }
            }
```
### OK。从android本地数据读取数据，并向服务器sqlserver数据库插入数据的场景就完成了，是不是非常的简单呢。有空的话，可以自己试一试吧。欢迎批评指正！
