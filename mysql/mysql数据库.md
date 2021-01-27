# mysql数据库

## 基本操作

### mysql的登录和登出

```sql
mysql -u用户名 -p 输入密码
mysql [-h主机命 -p 端口] -u用户名 -p密码
```

#### mysql的常见命令

```sql
-- 查看当前所有的数据库
show database;
-- 打开指定的数据库
use 数据库名

-- 查看指定库的所有表
shoe tables from 表名

-- 创建表
create table 表名(
列名 列类型，
    ***
);
-- 查看表结构
desc 表名

-- 查看mysql版本
select version();
mysql --version
```

#### mysql语法规范

1. 不区分大小写 但是建议关键字大写 其他小写
2. 每条命令最好使用用分号结尾
3. 每条命令根据需要可以进行换行。
4. 注释



DQL 数据查询语言 查

DML 数据操作语言 增删改

DDL  数据定义语言 库和表的定义

TCL 数据控制语言 事务相关的操作



### 实验数据

myemployees.sql

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : mysql0815
 Source Server Type    : MySQL
 Source Server Version : 50562
 Source Host           : localhost:3306
 Source Schema         : myemployees

 Target Server Type    : MySQL
 Target Server Version : 50562
 File Encoding         : 65001

 Date: 06/11/2019 16:25:51
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for departments
-- ----------------------------
DROP TABLE IF EXISTS `departments`;
CREATE TABLE `departments`  (
  `department_id` int(4) NOT NULL AUTO_INCREMENT,
  `department_name` varchar(3) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `manager_id` int(6) NULL DEFAULT NULL,
  `location_id` int(4) NULL DEFAULT NULL,
  PRIMARY KEY (`department_id`) USING BTREE,
  INDEX `loc_id_fk`(`location_id`) USING BTREE,
  CONSTRAINT `loc_id_fk` FOREIGN KEY (`location_id`) REFERENCES `locations` (`location_id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE = InnoDB AUTO_INCREMENT = 271 CHARACTER SET = gb2312 COLLATE = gb2312_chinese_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of departments
-- ----------------------------
INSERT INTO `departments` VALUES (10, 'Adm', 200, 1700);
INSERT INTO `departments` VALUES (20, 'Mar', 201, 1800);
INSERT INTO `departments` VALUES (30, 'Pur', 114, 1700);
INSERT INTO `departments` VALUES (40, 'Hum', 203, 2400);
INSERT INTO `departments` VALUES (50, 'Shi', 121, 1500);
INSERT INTO `departments` VALUES (60, 'IT', 103, 1400);
INSERT INTO `departments` VALUES (70, 'Pub', 204, 2700);
INSERT INTO `departments` VALUES (80, 'Sal', 145, 2500);
INSERT INTO `departments` VALUES (90, 'Exe', 100, 1700);
INSERT INTO `departments` VALUES (100, 'Fin', 108, 1700);
INSERT INTO `departments` VALUES (110, 'Acc', 205, 1700);
INSERT INTO `departments` VALUES (120, 'Tre', NULL, 1700);
INSERT INTO `departments` VALUES (130, 'Cor', NULL, 1700);
INSERT INTO `departments` VALUES (140, 'Con', NULL, 1700);
INSERT INTO `departments` VALUES (150, 'Sha', NULL, 1700);
INSERT INTO `departments` VALUES (160, 'Ben', NULL, 1700);
INSERT INTO `departments` VALUES (170, 'Man', NULL, 1700);
INSERT INTO `departments` VALUES (180, 'Con', NULL, 1700);
INSERT INTO `departments` VALUES (190, 'Con', NULL, 1700);
INSERT INTO `departments` VALUES (200, 'Ope', NULL, 1700);
INSERT INTO `departments` VALUES (210, 'IT ', NULL, 1700);
INSERT INTO `departments` VALUES (220, 'NOC', NULL, 1700);
INSERT INTO `departments` VALUES (230, 'IT ', NULL, 1700);
INSERT INTO `departments` VALUES (240, 'Gov', NULL, 1700);
INSERT INTO `departments` VALUES (250, 'Ret', NULL, 1700);
INSERT INTO `departments` VALUES (260, 'Rec', NULL, 1700);
INSERT INTO `departments` VALUES (270, 'Pay', NULL, 1700);

-- ----------------------------
-- Table structure for employees
-- ----------------------------
DROP TABLE IF EXISTS `employees`;
CREATE TABLE `employees`  (
  `employee_id` int(6) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(20) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `last_name` varchar(25) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `email` varchar(25) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `phone_number` varchar(20) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `job_id` varchar(10) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `salary` double(10, 2) NULL DEFAULT NULL,
  `commission_pct` double(4, 2) NULL DEFAULT NULL,
  `manager_id` int(6) NULL DEFAULT NULL,
  `department_id` int(4) NULL DEFAULT NULL,
  `hiredate` datetime NULL DEFAULT NULL,
  PRIMARY KEY (`employee_id`) USING BTREE,
  INDEX `dept_id_fk`(`department_id`) USING BTREE,
  INDEX `job_id_fk`(`job_id`) USING BTREE,
  CONSTRAINT `dept_id_fk` FOREIGN KEY (`department_id`) REFERENCES `departments` (`department_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `job_id_fk` FOREIGN KEY (`job_id`) REFERENCES `jobs` (`job_id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE = InnoDB AUTO_INCREMENT = 207 CHARACTER SET = gb2312 COLLATE = gb2312_chinese_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of employees
-- ----------------------------
INSERT INTO `employees` VALUES (100, 'Steven', 'K_ing', 'SKING', '515.123.4567', 'AD_PRES', 24000.00, NULL, NULL, 90, '1992-04-03 00:00:00');
INSERT INTO `employees` VALUES (101, 'Neena', 'Kochhar', 'NKOCHHAR', '515.123.4568', 'AD_VP', 17000.00, NULL, 100, 90, '1992-04-03 00:00:00');
INSERT INTO `employees` VALUES (102, 'Lex', 'De Haan', 'LDEHAAN', '515.123.4569', 'AD_VP', 17000.00, NULL, 100, 90, '1992-04-03 00:00:00');
INSERT INTO `employees` VALUES (103, 'Alexander', 'Hunold', 'AHUNOLD', '590.423.4567', 'IT_PROG', 9000.00, NULL, 102, 60, '1992-04-03 00:00:00');
INSERT INTO `employees` VALUES (104, 'Bruce', 'Ernst', 'BERNST', '590.423.4568', 'IT_PROG', 6000.00, NULL, 103, 60, '1992-04-03 00:00:00');
INSERT INTO `employees` VALUES (105, 'David', 'Austin', 'DAUSTIN', '590.423.4569', 'IT_PROG', 4800.00, NULL, 103, 60, '1998-03-03 00:00:00');
INSERT INTO `employees` VALUES (106, 'Valli', 'Pataballa', 'VPATABAL', '590.423.4560', 'IT_PROG', 4800.00, NULL, 103, 60, '1998-03-03 00:00:00');
INSERT INTO `employees` VALUES (107, 'Diana', 'Lorentz', 'DLORENTZ', '590.423.5567', 'IT_PROG', 4200.00, NULL, 103, 60, '1998-03-03 00:00:00');
INSERT INTO `employees` VALUES (108, 'Nancy', 'Greenberg', 'NGREENBE', '515.124.4569', 'FI_MGR', 12000.00, NULL, 101, 100, '1998-03-03 00:00:00');
INSERT INTO `employees` VALUES (109, 'Daniel', 'Faviet', 'DFAVIET', '515.124.4169', 'FI_ACCOUNT', 9000.00, NULL, 108, 100, '1998-03-03 00:00:00');
INSERT INTO `employees` VALUES (110, 'John', 'Chen', 'JCHEN', '515.124.4269', 'FI_ACCOUNT', 8200.00, NULL, 108, 100, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (111, 'Ismael', 'Sciarra', 'ISCIARRA', '515.124.4369', 'FI_ACCOUNT', 7700.00, NULL, 108, 100, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (112, 'Jose Manuel', 'Urman', 'JMURMAN', '515.124.4469', 'FI_ACCOUNT', 7800.00, NULL, 108, 100, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (113, 'Luis', 'Popp', 'LPOPP', '515.124.4567', 'FI_ACCOUNT', 6900.00, NULL, 108, 100, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (114, 'Den', 'Raphaely', 'DRAPHEAL', '515.127.4561', 'PU_MAN', 11000.00, NULL, 100, 30, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (115, 'Alexander', 'Khoo', 'AKHOO', '515.127.4562', 'PU_CLERK', 3100.00, NULL, 114, 30, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (116, 'Shelli', 'Baida', 'SBAIDA', '515.127.4563', 'PU_CLERK', 2900.00, NULL, 114, 30, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (117, 'Sigal', 'Tobias', 'STOBIAS', '515.127.4564', 'PU_CLERK', 2800.00, NULL, 114, 30, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (118, 'Guy', 'Himuro', 'GHIMURO', '515.127.4565', 'PU_CLERK', 2600.00, NULL, 114, 30, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (119, 'Karen', 'Colmenares', 'KCOLMENA', '515.127.4566', 'PU_CLERK', 2500.00, NULL, 114, 30, '2000-09-09 00:00:00');
INSERT INTO `employees` VALUES (120, 'Matthew', 'Weiss', 'MWEISS', '650.123.1234', 'ST_MAN', 8000.00, NULL, 100, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (121, 'Adam', 'Fripp', 'AFRIPP', '650.123.2234', 'ST_MAN', 8200.00, NULL, 100, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (122, 'Payam', 'Kaufling', 'PKAUFLIN', '650.123.3234', 'ST_MAN', 7900.00, NULL, 100, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (123, 'Shanta', 'Vollman', 'SVOLLMAN', '650.123.4234', 'ST_MAN', 6500.00, NULL, 100, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (124, 'Kevin', 'Mourgos', 'KMOURGOS', '650.123.5234', 'ST_MAN', 5800.00, NULL, 100, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (125, 'Julia', 'Nayer', 'JNAYER', '650.124.1214', 'ST_CLERK', 3200.00, NULL, 120, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (126, 'Irene', 'Mikkilineni', 'IMIKKILI', '650.124.1224', 'ST_CLERK', 2700.00, NULL, 120, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (127, 'James', 'Landry', 'JLANDRY', '650.124.1334', 'ST_CLERK', 2400.00, NULL, 120, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (128, 'Steven', 'Markle', 'SMARKLE', '650.124.1434', 'ST_CLERK', 2200.00, NULL, 120, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (129, 'Laura', 'Bissot', 'LBISSOT', '650.124.5234', 'ST_CLERK', 3300.00, NULL, 121, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (130, 'Mozhe', 'Atkinson', 'MATKINSO', '650.124.6234', 'ST_CLERK', 2800.00, NULL, 121, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (131, 'James', 'Marlow', 'JAMRLOW', '650.124.7234', 'ST_CLERK', 2500.00, NULL, 121, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (132, 'TJ', 'Olson', 'TJOLSON', '650.124.8234', 'ST_CLERK', 2100.00, NULL, 121, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (133, 'Jason', 'Mallin', 'JMALLIN', '650.127.1934', 'ST_CLERK', 3300.00, NULL, 122, 50, '2004-02-06 00:00:00');
INSERT INTO `employees` VALUES (134, 'Michael', 'Rogers', 'MROGERS', '650.127.1834', 'ST_CLERK', 2900.00, NULL, 122, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (135, 'Ki', 'Gee', 'KGEE', '650.127.1734', 'ST_CLERK', 2400.00, NULL, 122, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (136, 'Hazel', 'Philtanker', 'HPHILTAN', '650.127.1634', 'ST_CLERK', 2200.00, NULL, 122, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (137, 'Renske', 'Ladwig', 'RLADWIG', '650.121.1234', 'ST_CLERK', 3600.00, NULL, 123, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (138, 'Stephen', 'Stiles', 'SSTILES', '650.121.2034', 'ST_CLERK', 3200.00, NULL, 123, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (139, 'John', 'Seo', 'JSEO', '650.121.2019', 'ST_CLERK', 2700.00, NULL, 123, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (140, 'Joshua', 'Patel', 'JPATEL', '650.121.1834', 'ST_CLERK', 2500.00, NULL, 123, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (141, 'Trenna', 'Rajs', 'TRAJS', '650.121.8009', 'ST_CLERK', 3500.00, NULL, 124, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (142, 'Curtis', 'Davies', 'CDAVIES', '650.121.2994', 'ST_CLERK', 3100.00, NULL, 124, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (143, 'Randall', 'Matos', 'RMATOS', '650.121.2874', 'ST_CLERK', 2600.00, NULL, 124, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (144, 'Peter', 'Vargas', 'PVARGAS', '650.121.2004', 'ST_CLERK', 2500.00, NULL, 124, 50, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (145, 'John', 'Russell', 'JRUSSEL', '011.44.1344.429268', 'SA_MAN', 14000.00, 0.40, 100, 80, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (146, 'Karen', 'Partners', 'KPARTNER', '011.44.1344.467268', 'SA_MAN', 13500.00, 0.30, 100, 80, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (147, 'Alberto', 'Errazuriz', 'AERRAZUR', '011.44.1344.429278', 'SA_MAN', 12000.00, 0.30, 100, 80, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (148, 'Gerald', 'Cambrault', 'GCAMBRAU', '011.44.1344.619268', 'SA_MAN', 11000.00, 0.30, 100, 80, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (149, 'Eleni', 'Zlotkey', 'EZLOTKEY', '011.44.1344.429018', 'SA_MAN', 10500.00, 0.20, 100, 80, '2002-12-23 00:00:00');
INSERT INTO `employees` VALUES (150, 'Peter', 'Tucker', 'PTUCKER', '011.44.1344.129268', 'SA_REP', 10000.00, 0.30, 145, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (151, 'David', 'Bernstein', 'DBERNSTE', '011.44.1344.345268', 'SA_REP', 9500.00, 0.25, 145, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (152, 'Peter', 'Hall', 'PHALL', '011.44.1344.478968', 'SA_REP', 9000.00, 0.25, 145, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (153, 'Christopher', 'Olsen', 'COLSEN', '011.44.1344.498718', 'SA_REP', 8000.00, 0.20, 145, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (154, 'Nanette', 'Cambrault', 'NCAMBRAU', '011.44.1344.987668', 'SA_REP', 7500.00, 0.20, 145, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (155, 'Oliver', 'Tuvault', 'OTUVAULT', '011.44.1344.486508', 'SA_REP', 7000.00, 0.15, 145, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (156, 'Janette', 'K_ing', 'JKING', '011.44.1345.429268', 'SA_REP', 10000.00, 0.35, 146, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (157, 'Patrick', 'Sully', 'PSULLY', '011.44.1345.929268', 'SA_REP', 9500.00, 0.35, 146, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (158, 'Allan', 'McEwen', 'AMCEWEN', '011.44.1345.829268', 'SA_REP', 9000.00, 0.35, 146, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (159, 'Lindsey', 'Smith', 'LSMITH', '011.44.1345.729268', 'SA_REP', 8000.00, 0.30, 146, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (160, 'Louise', 'Doran', 'LDORAN', '011.44.1345.629268', 'SA_REP', 7500.00, 0.30, 146, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (161, 'Sarath', 'Sewall', 'SSEWALL', '011.44.1345.529268', 'SA_REP', 7000.00, 0.25, 146, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (162, 'Clara', 'Vishney', 'CVISHNEY', '011.44.1346.129268', 'SA_REP', 10500.00, 0.25, 147, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (163, 'Danielle', 'Greene', 'DGREENE', '011.44.1346.229268', 'SA_REP', 9500.00, 0.15, 147, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (164, 'Mattea', 'Marvins', 'MMARVINS', '011.44.1346.329268', 'SA_REP', 7200.00, 0.10, 147, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (165, 'David', 'Lee', 'DLEE', '011.44.1346.529268', 'SA_REP', 6800.00, 0.10, 147, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (166, 'Sundar', 'Ande', 'SANDE', '011.44.1346.629268', 'SA_REP', 6400.00, 0.10, 147, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (167, 'Amit', 'Banda', 'ABANDA', '011.44.1346.729268', 'SA_REP', 6200.00, 0.10, 147, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (168, 'Lisa', 'Ozer', 'LOZER', '011.44.1343.929268', 'SA_REP', 11500.00, 0.25, 148, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (169, 'Harrison', 'Bloom', 'HBLOOM', '011.44.1343.829268', 'SA_REP', 10000.00, 0.20, 148, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (170, 'Tayler', 'Fox', 'TFOX', '011.44.1343.729268', 'SA_REP', 9600.00, 0.20, 148, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (171, 'William', 'Smith', 'WSMITH', '011.44.1343.629268', 'SA_REP', 7400.00, 0.15, 148, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (172, 'Elizabeth', 'Bates', 'EBATES', '011.44.1343.529268', 'SA_REP', 7300.00, 0.15, 148, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (173, 'Sundita', 'Kumar', 'SKUMAR', '011.44.1343.329268', 'SA_REP', 6100.00, 0.10, 148, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (174, 'Ellen', 'Abel', 'EABEL', '011.44.1644.429267', 'SA_REP', 11000.00, 0.30, 149, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (175, 'Alyssa', 'Hutton', 'AHUTTON', '011.44.1644.429266', 'SA_REP', 8800.00, 0.25, 149, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (176, 'Jonathon', 'Taylor', 'JTAYLOR', '011.44.1644.429265', 'SA_REP', 8600.00, 0.20, 149, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (177, 'Jack', 'Livingston', 'JLIVINGS', '011.44.1644.429264', 'SA_REP', 8400.00, 0.20, 149, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (178, 'Kimberely', 'Grant', 'KGRANT', '011.44.1644.429263', 'SA_REP', 7000.00, 0.15, 149, NULL, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (179, 'Charles', 'Johnson', 'CJOHNSON', '011.44.1644.429262', 'SA_REP', 6200.00, 0.10, 149, 80, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (180, 'Winston', 'Taylor', 'WTAYLOR', '650.507.9876', 'SH_CLERK', 3200.00, NULL, 120, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (181, 'Jean', 'Fleaur', 'JFLEAUR', '650.507.9877', 'SH_CLERK', 3100.00, NULL, 120, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (182, 'Martha', 'Sullivan', 'MSULLIVA', '650.507.9878', 'SH_CLERK', 2500.00, NULL, 120, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (183, 'Girard', 'Geoni', 'GGEONI', '650.507.9879', 'SH_CLERK', 2800.00, NULL, 120, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (184, 'Nandita', 'Sarchand', 'NSARCHAN', '650.509.1876', 'SH_CLERK', 4200.00, NULL, 121, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (185, 'Alexis', 'Bull', 'ABULL', '650.509.2876', 'SH_CLERK', 4100.00, NULL, 121, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (186, 'Julia', 'Dellinger', 'JDELLING', '650.509.3876', 'SH_CLERK', 3400.00, NULL, 121, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (187, 'Anthony', 'Cabrio', 'ACABRIO', '650.509.4876', 'SH_CLERK', 3000.00, NULL, 121, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (188, 'Kelly', 'Chung', 'KCHUNG', '650.505.1876', 'SH_CLERK', 3800.00, NULL, 122, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (189, 'Jennifer', 'Dilly', 'JDILLY', '650.505.2876', 'SH_CLERK', 3600.00, NULL, 122, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (190, 'Timothy', 'Gates', 'TGATES', '650.505.3876', 'SH_CLERK', 2900.00, NULL, 122, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (191, 'Randall', 'Perkins', 'RPERKINS', '650.505.4876', 'SH_CLERK', 2500.00, NULL, 122, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (192, 'Sarah', 'Bell', 'SBELL', '650.501.1876', 'SH_CLERK', 4000.00, NULL, 123, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (193, 'Britney', 'Everett', 'BEVERETT', '650.501.2876', 'SH_CLERK', 3900.00, NULL, 123, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (194, 'Samuel', 'McCain', 'SMCCAIN', '650.501.3876', 'SH_CLERK', 3200.00, NULL, 123, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (195, 'Vance', 'Jones', 'VJONES', '650.501.4876', 'SH_CLERK', 2800.00, NULL, 123, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (196, 'Alana', 'Walsh', 'AWALSH', '650.507.9811', 'SH_CLERK', 3100.00, NULL, 124, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (197, 'Kevin', 'Feeney', 'KFEENEY', '650.507.9822', 'SH_CLERK', 3000.00, NULL, 124, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (198, 'Donald', 'OConnell', 'DOCONNEL', '650.507.9833', 'SH_CLERK', 2600.00, NULL, 124, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (199, 'Douglas', 'Grant', 'DGRANT', '650.507.9844', 'SH_CLERK', 2600.00, NULL, 124, 50, '2014-03-05 00:00:00');
INSERT INTO `employees` VALUES (200, 'Jennifer', 'Whalen', 'JWHALEN', '515.123.4444', 'AD_ASST', 4400.00, NULL, 101, 10, '2016-03-03 00:00:00');
INSERT INTO `employees` VALUES (201, 'Michael', 'Hartstein', 'MHARTSTE', '515.123.5555', 'MK_MAN', 13000.00, NULL, 100, 20, '2016-03-03 00:00:00');
INSERT INTO `employees` VALUES (202, 'Pat', 'Fay', 'PFAY', '603.123.6666', 'MK_REP', 6000.00, NULL, 201, 20, '2016-03-03 00:00:00');
INSERT INTO `employees` VALUES (203, 'Susan', 'Mavris', 'SMAVRIS', '515.123.7777', 'HR_REP', 6500.00, NULL, 101, 40, '2016-03-03 00:00:00');
INSERT INTO `employees` VALUES (204, 'Hermann', 'Baer', 'HBAER', '515.123.8888', 'PR_REP', 10000.00, NULL, 101, 70, '2016-03-03 00:00:00');
INSERT INTO `employees` VALUES (205, 'Shelley', 'Higgins', 'SHIGGINS', '515.123.8080', 'AC_MGR', 12000.00, NULL, 101, 110, '2016-03-03 00:00:00');
INSERT INTO `employees` VALUES (206, 'William', 'Gietz', 'WGIETZ', '515.123.8181', 'AC_ACCOUNT', 8300.00, NULL, 205, 110, '2016-03-03 00:00:00');

-- ----------------------------
-- Table structure for job_grades
-- ----------------------------
DROP TABLE IF EXISTS `job_grades`;
CREATE TABLE `job_grades`  (
  `grade_level` varchar(3) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `lowest_sal` int(11) NULL DEFAULT NULL,
  `highest_sal` int(11) NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of job_grades
-- ----------------------------
INSERT INTO `job_grades` VALUES ('A', 1000, 2999);
INSERT INTO `job_grades` VALUES ('B', 3000, 5999);
INSERT INTO `job_grades` VALUES ('C', 6000, 9999);
INSERT INTO `job_grades` VALUES ('D', 10000, 14999);
INSERT INTO `job_grades` VALUES ('E', 15000, 24999);
INSERT INTO `job_grades` VALUES ('F', 25000, 40000);

-- ----------------------------
-- Table structure for jobs
-- ----------------------------
DROP TABLE IF EXISTS `jobs`;
CREATE TABLE `jobs`  (
  `job_id` varchar(10) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NOT NULL,
  `job_title` varchar(35) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `min_salary` int(6) NULL DEFAULT NULL,
  `max_salary` int(6) NULL DEFAULT NULL,
  PRIMARY KEY (`job_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = gb2312 COLLATE = gb2312_chinese_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of jobs
-- ----------------------------
INSERT INTO `jobs` VALUES ('AC_ACCOUNT', 'Public Accountant', 4200, 9000);
INSERT INTO `jobs` VALUES ('AC_MGR', 'Accounting Manager', 8200, 16000);
INSERT INTO `jobs` VALUES ('AD_ASST', 'Administration Assistant', 3000, 6000);
INSERT INTO `jobs` VALUES ('AD_PRES', 'President', 20000, 40000);
INSERT INTO `jobs` VALUES ('AD_VP', 'Administration Vice President', 15000, 30000);
INSERT INTO `jobs` VALUES ('FI_ACCOUNT', 'Accountant', 4200, 9000);
INSERT INTO `jobs` VALUES ('FI_MGR', 'Finance Manager', 8200, 16000);
INSERT INTO `jobs` VALUES ('HR_REP', 'Human Resources Representative', 4000, 9000);
INSERT INTO `jobs` VALUES ('IT_PROG', 'Programmer', 4000, 10000);
INSERT INTO `jobs` VALUES ('MK_MAN', 'Marketing Manager', 9000, 15000);
INSERT INTO `jobs` VALUES ('MK_REP', 'Marketing Representative', 4000, 9000);
INSERT INTO `jobs` VALUES ('PR_REP', 'Public Relations Representative', 4500, 10500);
INSERT INTO `jobs` VALUES ('PU_CLERK', 'Purchasing Clerk', 2500, 5500);
INSERT INTO `jobs` VALUES ('PU_MAN', 'Purchasing Manager', 8000, 15000);
INSERT INTO `jobs` VALUES ('SA_MAN', 'Sales Manager', 10000, 20000);
INSERT INTO `jobs` VALUES ('SA_REP', 'Sales Representative', 6000, 12000);
INSERT INTO `jobs` VALUES ('SH_CLERK', 'Shipping Clerk', 2500, 5500);
INSERT INTO `jobs` VALUES ('ST_CLERK', 'Stock Clerk', 2000, 5000);
INSERT INTO `jobs` VALUES ('ST_MAN', 'Stock Manager', 5500, 8500);

-- ----------------------------
-- Table structure for locations
-- ----------------------------
DROP TABLE IF EXISTS `locations`;
CREATE TABLE `locations`  (
  `location_id` int(11) NOT NULL AUTO_INCREMENT,
  `street_address` varchar(40) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `postal_code` varchar(12) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `city` varchar(30) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `state_province` varchar(25) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  `country_id` varchar(2) CHARACTER SET gb2312 COLLATE gb2312_chinese_ci NULL DEFAULT NULL,
  PRIMARY KEY (`location_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3201 CHARACTER SET = gb2312 COLLATE = gb2312_chinese_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of locations
-- ----------------------------
INSERT INTO `locations` VALUES (1000, '1297 Via Cola di Rie', '00989', 'Roma', NULL, 'IT');
INSERT INTO `locations` VALUES (1100, '93091 Calle della Testa', '10934', 'Venice', NULL, 'IT');
INSERT INTO `locations` VALUES (1200, '2017 Shinjuku-ku', '1689', 'Tokyo', 'Tokyo Prefecture', 'JP');
INSERT INTO `locations` VALUES (1300, '9450 Kamiya-cho', '6823', 'Hiroshima', NULL, 'JP');
INSERT INTO `locations` VALUES (1400, '2014 Jabberwocky Rd', '26192', 'Southlake', 'Texas', 'US');
INSERT INTO `locations` VALUES (1500, '2011 Interiors Blvd', '99236', 'South San Francisco', 'California', 'US');
INSERT INTO `locations` VALUES (1600, '2007 Zagora St', '50090', 'South Brunswick', 'New Jersey', 'US');
INSERT INTO `locations` VALUES (1700, '2004 Charade Rd', '98199', 'Seattle', 'Washington', 'US');
INSERT INTO `locations` VALUES (1800, '147 Spadina Ave', 'M5V 2L7', 'Toronto', 'Ontario', 'CA');
INSERT INTO `locations` VALUES (1900, '6092 Boxwood St', 'YSW 9T2', 'Whitehorse', 'Yukon', 'CA');
INSERT INTO `locations` VALUES (2000, '40-5-12 Laogianggen', '190518', 'Beijing', NULL, 'CN');
INSERT INTO `locations` VALUES (2100, '1298 Vileparle (E)', '490231', 'Bombay', 'Maharashtra', 'IN');
INSERT INTO `locations` VALUES (2200, '12-98 Victoria Street', '2901', 'Sydney', 'New South Wales', 'AU');
INSERT INTO `locations` VALUES (2300, '198 Clementi North', '540198', 'Singapore', NULL, 'SG');
INSERT INTO `locations` VALUES (2400, '8204 Arthur St', NULL, 'London', NULL, 'UK');
INSERT INTO `locations` VALUES (2500, 'Magdalen Centre, The Oxford Science Park', 'OX9 9ZB', 'Oxford', 'Oxford', 'UK');
INSERT INTO `locations` VALUES (2600, '9702 Chester Road', '09629850293', 'Stretford', 'Manchester', 'UK');
INSERT INTO `locations` VALUES (2700, 'Schwanthalerstr. 7031', '80925', 'Munich', 'Bavaria', 'DE');
INSERT INTO `locations` VALUES (2800, 'Rua Frei Caneca 1360 ', '01307-002', 'Sao Paulo', 'Sao Paulo', 'BR');
INSERT INTO `locations` VALUES (2900, '20 Rue des Corps-Saints', '1730', 'Geneva', 'Geneve', 'CH');
INSERT INTO `locations` VALUES (3000, 'Murtenstrasse 921', '3095', 'Bern', 'BE', 'CH');
INSERT INTO `locations` VALUES (3100, 'Pieter Breughelstraat 837', '3029SK', 'Utrecht', 'Utrecht', 'NL');
INSERT INTO `locations` VALUES (3200, 'Mariano Escobedo 9991', '11932', 'Mexico City', 'Distrito Federal,', 'MX');

SET FOREIGN_KEY_CHECKS = 1;


```

girl.sql文件

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : mysql0815
 Source Server Type    : MySQL
 Source Server Version : 50562
 Source Host           : localhost:3306
 Source Schema         : girls

 Target Server Type    : MySQL
 Target Server Version : 50562
 File Encoding         : 65001

 Date: 06/11/2019 16:26:51
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for admin
-- ----------------------------
DROP TABLE IF EXISTS `admin`;
CREATE TABLE `admin`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `password` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of admin
-- ----------------------------
INSERT INTO `admin` VALUES (1, 'john', '8888');
INSERT INTO `admin` VALUES (2, 'lyt', '6666');

-- ----------------------------
-- Table structure for beauty
-- ----------------------------
DROP TABLE IF EXISTS `beauty`;
CREATE TABLE `beauty`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `sex` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT 'Ů',
  `borndate` datetime NULL DEFAULT '1987-01-01 00:00:00',
  `phone` varchar(11) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `photo` blob NULL,
  `boyfriend_id` int(11) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 13 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of beauty
-- ----------------------------
INSERT INTO `beauty` VALUES (1, '柳岩', '女', '1988-02-03 00:00:00', '18209876577', NULL, 8);
INSERT INTO `beauty` VALUES (2, '苍老师', '女', '1987-12-30 00:00:00', '18219876577', NULL, 9);
INSERT INTO `beauty` VALUES (3, 'Angelababy', '女', '1989-02-03 00:00:00', '18209876567', NULL, 3);
INSERT INTO `beauty` VALUES (4, '热巴', '女', '1993-02-03 00:00:00', '18209876579', NULL, 2);
INSERT INTO `beauty` VALUES (5, '周冬雨', '女', '1992-02-03 00:00:00', '18209179577', NULL, 9);
INSERT INTO `beauty` VALUES (6, '周芷若', '女', '1988-02-03 00:00:00', '18209876577', NULL, 1);
INSERT INTO `beauty` VALUES (7, '岳灵珊', '女', '1987-12-30 00:00:00', '18219876577', NULL, 9);
INSERT INTO `beauty` VALUES (8, '小昭', '女', '1989-02-03 00:00:00', '18209876567', NULL, 1);
INSERT INTO `beauty` VALUES (9, '双儿', '女', '1993-02-03 00:00:00', '18209876579', NULL, 9);
INSERT INTO `beauty` VALUES (10, '王语嫣', '女', '1992-02-03 00:00:00', '18209179577', NULL, 4);
INSERT INTO `beauty` VALUES (11, '夏雪', '女', '1993-02-03 00:00:00', '18209876579', NULL, 9);
INSERT INTO `beauty` VALUES (12, '赵敏', '女', '1992-02-03 00:00:00', '18209179577', NULL, 1);

-- ----------------------------
-- Table structure for boys
-- ----------------------------
DROP TABLE IF EXISTS `boys`;
CREATE TABLE `boys`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `boyName` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `userCP` int(11) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of boys
-- ----------------------------
INSERT INTO `boys` VALUES (1, '张无忌', 100);
INSERT INTO `boys` VALUES (2, '鹿晗', 800);
INSERT INTO `boys` VALUES (3, '黄晓', 50);
INSERT INTO `boys` VALUES (4, '段誉', 300);

SET FOREIGN_KEY_CHECKS = 1;


```

job_grades.sql

```sql
USE myemployees;
CREATE TABLE job_grades(
	grade_level VARCHAR(3),
	lowest_sal  INT,
	highest_sal INT
);

INSERT INTO job_grades
VALUES ('A', 1000, 2999);

INSERT INTO job_grades
VALUES ('B', 3000, 5999);

INSERT INTO job_grades
VALUES('C', 6000, 9999);

INSERT INTO job_grades
VALUES('D', 10000, 14999);

INSERT INTO job_grades
VALUES('E', 15000, 24999);

INSERT INTO job_grades
VALUES('F', 25000, 40000);


```

## 单表操作

### 普通查询

```sql
-- 查询表中单个字段
SELECT last_name FROM employees;

-- 查询表中的多个字段
SELECT last_name,salary,email FROM employees;

-- 查询全部字段
SELECT * FROM employees;

-- 字段带着重号 可以去掉 代表这是一个字段名称
SELECT `last_name`,`salary`,`email` FROM employees;



-- 查询常量值 查询的结果和字段都是本身
SELECT 100;

-- 查询表达式
SELECT 100*98;

-- 查询函数
SELECT VERSION();

# 为字段起别名

/*
① 便于理解
② 重名的时候 可以区分 
as 可以省略
*/
SELECT VERSION() AS `mysql_version`; 

-- 查询salary 输出为 out put 出现特殊符号的时候需要使用引号

SELECT salary AS "out put" FROM employees;

-- 去重

SELECT DISTINCT department_id FROM employees;

# +的作用
-- 查询员工名和员工姓为一个字
/**
java 中的+ 
运算符
连接符 只要有一个为字符串 就会拼接
在mysql中+只是运算符
在mysql中如果有一个字段为字符串 就会试图进行转换如果转换成功就按照字符串
代表的数值进行运算 否则就转换成0 只要其中的一方为null 结果就是null
字符串利用concat进行拼接

*/
SELECT CONCAT(last_name,' ',first_name) AS 姓名 FROM employees;



DESCRIBE `departments`;

SELECT DISTINCT job_id FROM employees;

-- ifnull函数
SELECT IFNULL(commission_pct,0)AS '奖金率' FROM employees;



SELECT
  CONCAT(
    first_name,
    ',',
    last_name,
    ',',
    IFNULL(commission_pct, 0)
  ) AS info
FROM
  employees;


```

### 带条件查询

```sql
/*
条件查询
语法 select 查询列表 from 表名 where 筛选条件;

分类：
	1、 条件表达式筛选 条件运算符 > = < <> <= >=
	2、 按逻辑表达式筛选 使用的是逻辑运算符  and or not
	&& 和 and 如果两个条件为true 结果为true 反之为false
	！和 not 取反
	or 只要有一个为true 就位true
	3、 模糊查询 like in between and is null
	
	
*/
-- 条件表达式查询
 SELECT
  *
FROM
  employees
WHERE salary > 12000;

# 查询部门标号不等于90号的员工名和部门编号
 SELECT
  last_name,
  department_id
FROM
  employees
WHERE department_id <> 90;

# 按照逻辑表达式筛选
 # 查询工资大于10000小于20000的员工信息
 SELECT
  last_name,
  salary,
  commission_pct
FROM
  employees
WHERE salary > 10000
  AND salary < 20000;
 
 
 -- 模糊查询
 -- like
 -- 查询员工名中包含支付a的员工信息
 -- 一般和通配符使用 
 /*
 % like 包含0-n个字符 
 _ 任意单个字符
 
 字符的转义可以使用\
 */ 
 SELECT * FROM employees WHERE last_name LIKE '%a%';
 
 /*
 between and
 查询工资在10000-20000之间的员工
 包含左右区间 但是不能颠倒顺序
 */
 
 SELECT * FROM employees WHERE salary BETWEEN 10000 AND	20000;
 
 /*
in
提高简洁度
in里面的类型必须是同类型 或者类型兼容的
不支持%等通配符
查询员工的工种编号是 在 AD_VP PU_MAN
 */
 SELECT * FROM employees WHERE job_id IN('AD_VP', 'PU_MAN');
 
 
/*
isnull

查询没有奖金率的员工

*/

SELECT * FROM employees WHERE	commission_pct IS NULL;
SELECT * FROM employees WHERE	commission_pct IS NOT NULL;

/*
安全等于 <=>
可以判断null值
也可以判断数值 
is null 只能判断null <=>可以判断null和数值 但是
可读性不好 因此建议使用is null 和 =
*/

SELECT * FROM employees WHERE commission_pct <=> NULL;
SELECT * FROM employees WHERE commission_pct <=> 0.2;
```

### 排序

```sql
/*
排序功能
语法 select 查询列表 from 表 where 筛选条件 order by 排序列表 [asc|desc]

查询员工信息 要求工资从高到低
一般情况下order by 是放在最后面的 limit 除外。
*/


SELECT
  *
FROM
  employees
ORDER BY salary DESC;

/*查询部门编号》90 按照入职时间进行排序*/
SELECT
  *
FROM
  employees
WHERE employee_id > 90
ORDER BY hiredate;

/*按照表达式排序 按照年薪排序  */
SELECT
  *,
  salary * 12 * (1+ IFNULL(commission_pct, 0)) AS year_salary
FROM
  employees
ORDER BY
salary * 12 * (1+ IFNULL(commission_pct, 0));
  /*按照别名排序 年薪降序*/
  SELECT
    *,
    salary * 12 * (1+ IFNULL(commission_pct, 0)) AS year_salary
  FROM
    employees
  ORDER BY year_salary DESC;

/*按照员工姓名的长度进行排序*/
SELECT
  LENGTH(last_name) AS name_length,
  last_name
FROM
  employees
ORDER BY name_length DESC;


/*多个字段的排序*/

SELECT
  LENGTH(last_name) AS name_length,
  last_name,salary
FROM
  employees
ORDER BY name_length DESC,salary DESC;


```

### 常见函数



#### 单行函数

```sql
-- 进阶4 常见函数

/*
功能 类似于java的方法 将我们的一组逻辑语句封装在函数内
对外暴露方法名
调用：slect 函数名(参数列表) from 表名 
特点 ① 函数名 ② 函数功能
分类 单行函数  concat length 处理
     分组函数	统计函数
*/

/*单行函数：

字符函数
数学函数
日期函数
其他函数
流程控制函数
*/

/*字符函数

length
*/
SELECT LENGTH('1222');
SELECT LENGTH("张三丰"); # utf8 字符集 占据3个字节

# concat 拼接函数
SELECT CONCAT(first_name,last_name) FROM employees;

# upper lower 大小写转换

SELECT CONCAT(UPPER(first_name)," ",LOWER(last_name)) FROM employees;


# substr substring 截取字符串 开始位置从1
SELECT SUBSTR('李莫愁',2) out_put;
# 截取的为字符的长度
SELECT SUBSTR('李莫愁',1,1) out_put;

# instr 
SELECT INSTR('杨不悔爱上了殷六侠','殷六侠') AS output;

# trim 去空格
SELECT TRIM('   heloo   ') AS output;

SELECT TRIM('a' FROM 'aaaaahhhhhaaaa') AS output;

# lpad 用指定的字符实现左填充 rpad 实现右填充

SELECT LPAD('殷素素',10,'*') AS output;


# replace
SELECT REPLACE("周芷若爱上了张无忌",'周芷若','赵敏') AS  output;


# 数学函数

#  round 四舍五入
SELECT ROUND(1.45);
SELECT ROUND(1.654,2); #保留两位小数

-- 向上取整
SELECT CEIL(2.2);
 
-- 向下取整
SELECT FLOOR(2.5);


-- 日期函数
# now 返回当前日期+时间
SELECT NOW();

# 返回当前日期
SELECT CURDATE();

# 返回当前时间
SELECT CURTIME();

# 在日期中获取年月日

SELECT YEAR(hiredate) AS `year`   FROM employees;


# 格式化日期
SELECT employee_id,DATE_FORMAT(hiredate,'%Y年%m月%d日') FROM employees WHERE commission_pct IS NOT NULL;


/*其他函数*/

SELECT VERSION();


# 流程控制函数

# if 函数
SELECT IF(10>5,'大','小');


# case函数
/*
switch case 函数

mysql中 case 要判断字段或者表达式
when 常量1： then：要显示的值或者语句;
...
else 默认的语句
end


部门号=30 显示的工资为1.1被
部门号=40 。。。1.2
..=50   1.3
*/
SELECT department_id,salary 原始工资,
CASE department_id
WHEN 30 THEN salary*1.1
WHEN 40 THEN salary*1.2
WHEN 50 THEN salary*1.3
ELSE salary
END AS 新工资
FROM employees;

# case 函数的使用 类似多重if
/*

在mysql中 

作为一个字段显示
case
when 条件1 then 语句1
when 条件2 then 语句2
when 条件3 then 语句3
。。
else 要显示的值n或者语句n
end

如果工资>20000显示A级别
如果工资>15000 显示B级别
如果工资>10000 显示C级别

*/
SELECT salary,
CASE
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS `level`
FROM employees;


```

#### 分组函数

```sql
-- 分组喊你书

SELECT SUM(salary) FROM employees;
SELECT MAX(salary) FROM employees;
SELECT COUNT(salary) FROM employees;
/*
功能

可以与distinct搭配实现去重统计
*/

SELECT SUM(DISTINCT salary) FROM employees;
SELECT COUNT(DISTINCT salary) FROM employees;


# count函数
-- count(*)用于统计行数
SELECT COUNT(*) FROM employees;
-- 同理 加常量值 也是用于统计
SELECT COUNT(1) FROM employees;

-- 效率 INNODB 在MYISAM count(*)效率最高 在INNODB count(*)与cout(1)效率差不多 但是大于字段

-- 和分组函数一同查询的字段限制 一般要求的是group by的


```

```sql
# 进阶5 分组查询
/*

select 分组函数，列
 分组前筛选 原始表 where
 分组后筛选 使用筛选后的结果集 having
 分组函数作维筛选使用having
 
特点 1、分组查询的筛选条件分为两类
分组前筛选
分组后筛选 
支持多个字段分组，分组之间的顺序无影响 
*/


-- 插叙每个部门的平均工资
SELECT AVG(salary) FROM employees GROUP BY department_id;


SELECT MAX(salary),manager_id FROM employees WHERE commission_pct IS NOT NULL GROUP BY manager_id;

-- 查询部门的员工个数大于2

-- 先查询每个部门的员工数
-- 在根据每个部门的员工数进行筛选 使用having
SELECT COUNT(*),department_id FROM employees GROUP BY department_id HAVING COUNT(*)>2;


-- 查询每个工种有奖金的员工最高工资》12000的工种编号和最高工资

SELECT MAX(salary) AS max_salary,job_id FROM employees 
WHERE commission_pct IS NOT NULL
GROUP BY job_id
HAVING max_salary>12000;


-- 按照表达式或者函数分组

-- ① 查询姓名长度的员工个数
-- 按照员工的姓名长度进行分组，查询每一组的员工个数，筛选员工个数>5的有哪些 

SELECT COUNT(*) AS `count`,LENGTH(last_name) AS len_name FROM employees GROUP BY len_name HAVING COUNT(*)>5 ORDER BY COUNT(*) DESC;

```

## 多表操作

### 笛卡尔乘积现象

```sql
SELECT
  *
FROM
  beauty,
  boys;

/*
笛卡尔乘积现象 如果表1有m行，表2有n行 那么查询的结果是m*n行


内连接
	等值连接
	非等值连接
	自连接
外连接
	左外连接
	右外连接
	全外连接
交叉连接

*/
SELECT
  beauty.`name`,
  boys.`boyName`
FROM
  beauty,
  boys
WHERE beauty.`boyfriend_id` = boys.`id`;

/*
一、sql92标准
1 等值连接
*/
SELECT
  beauty.`name`,
  boys.`boyName`
FROM
  beauty,
  boys
WHERE beauty.`boyfriend_id` = boys.`id`;

SELECT
  last_name,
  department_name
FROM
  employees,
  departments
WHERE employees.`department_id` = departments.`department_id`;

# 为表起别名
 SELECT
  e.`last_name`,
  j.`job_id`,
  j.`job_title`
FROM
  employees e,
  jobs j
WHERE e.`job_id` = j.`job_id`;

-- 非等值连接
 select
  salary,
  grade_level
from
  employees e,
  job_grades g
where salary between g.`lowest_sal`
  and g.`highest_sal`;

/*
sql 99
select 查询列表
from 表1 别名 [连接类型]
join 表2 北冥
on 连接条件

内连接 inner
外连接 

左外
右外
全外
交叉连接 
*/
-- 查询员工名和部门名
 select
  last_name,
  department_name
from
  employees e
  inner join departments d
    on e.`department_id` = d.`department_id`;

-- 案例 查询那个部门的员工个数》3的部门名和员工个数 并按照个数降序
select
  department_name,
  count(*) as employee_num
from
  departments
  inner join employees
    on employees.`department_id` = departments.`department_id`
group by departments.`department_id`
having employee_num > 3
order by employee_num desc;


```

### 子查询

```sql
-- 内连接
SELECT beauty.`name`,beauty.`phone`,beauty.`boyfriend_id`,
boys.`boyName` FROM beauty JOIN boys WHERE 
beauty.`boyfriend_id`=boys.`id`;

-- 左外连接
 SELECT
  beauty.`name`,
  beauty.`phone`,
  beauty.`boyfriend_id`,
  boys.`boyName`
FROM
  beauty
  LEFT JOIN boys
ON beauty.`boyfriend_id` = boys.`id`;

-- 右外查询
 SELECT
  beauty.`name`,
  beauty.`phone`,
  beauty.`boyfriend_id`,
  boys.`boyName`
FROM
  beauty
  RIGHT JOIN boys
ON beauty.`boyfriend_id` = boys.`id`;
/*
子查询
子查询需要使用小括号
子查询一般放在条件的右侧
一般搭配标量子查询 一般搭配多行子查询使用



*/

# 1. 标量子查询

/*
谁的工资比Abel高
先查询Abel的工资
*/

SELECT salary FROM employees WHERE last_name='Abel';

SELECT
  last_name,
  salary
FROM
  employees
WHERE salary >
  (SELECT
    salary
  FROM
    employees
  WHERE last_name = 'Abel');
  
/*
案例2 返回job_id 与141号员工相同，salary比143号员工多的
员工 姓名 job——id 工资1
*/

SELECT job_id FROM employees WHERE employee_id = 141;
SELECT salary FROM employees WHERE employee_id = 143;

SELECT
  last_name,
  job_id,
  salary
FROM
  employees
WHERE job_id =
  (SELECT
    job_id
  FROM
    employees
  WHERE employee_id = 141)
  AND salary >
  (SELECT
    salary
  FROM
    employees
  WHERE employee_id = 143);
  
  
  # 案例3 返回 公式工资最少的运功的last_name,job_id 和salary
  
  SELECT MIN(salary) FROM employees;
  
  SELECT last_name,job_id,salary FROM employees
  WHERE salary=(
  SELECT MIN(salary) FROM employees
  );
  
  
  # 查询最低工资大于50号部门的最低工资的部门id哈最低工资
  SELECT MIN(salary) FROM employees WHERE department_id  = 50;
  
SELECT
  MIN(salary) min_salary,
  department_id
FROM
  employees
GROUP BY department_id
HAVING  min_salary >
  (SELECT
    MIN(salary)
  FROM
    employees
  WHERE department_id = 50);

-- 行子查询 一行多列 多行多列

-- 查询员工编号以及工资最高的员工信息。

SELECT COUNT(*),department_id FROM employees GROUP BY department_id;

SELECT department_name FROM departments INNER JOIN employees ON employees.`employee_id`=102 AND departments.`department_id`=employees.`department_id`;

-- 联合查询
/*
union 
多条查询语句的结果合并成一个结果
查询语句1
union 
查询语句2


应用场景

要查询的结果来自于多个表 
*/

# 查询部门编号>90 或者邮箱包含a的员工信息

SELECT * FROM employees WHERE department_id>90
UNION
SELECT * FROM employees WHERE email LIKE '%a%';
```

