#!/usr/bin/env python

import numpy as np
import matplotlib.pyplot as plt
import psycopg2
import getpass
import os
import time
import subprocess

from os.path import expanduser
from operator import sub

Uservar=getpass.getuser()
home=expanduser("~")

DirvarTpch=raw_input("Greetings! Please type the name of the directory that you want to save TPCH: ")

pathcustomer=os.path.join(home,DirvarTpch,'pg-tpch','dbgen','customer.tbl')
pathnation=os.path.join(home,DirvarTpch,'pg-tpch','dbgen','nation.tbl')
pathpartsupp=os.path.join(home,DirvarTpch,'pg-tpch','dbgen','partsupp.tbl')
pathregion=os.path.join(home,DirvarTpch,'pg-tpch','dbgen','region.tbl')
pathlineitem=os.path.join(home,DirvarTpch,'pg-tpch','dbgen','lineitem.tbl')
pathorders=os.path.join(home,DirvarTpch,'pg-tpch','dbgen','orders.tbl')
pathpart=os.path.join(home,DirvarTpch,'pg-tpch','dbgen','part.tbl')
pathsupplier=os.path.join(home,DirvarTpch,'pg-tpch','dbgen','supplier.tbl')

while os.path.exists(os.path.join(home,DirvarTpch))==True:
    DirvarTpch=raw_input("The folder already exists, please type another name: ")

os.chdir(home)
os.mkdir(DirvarTpch)
os.chdir(DirvarTpch)
os.system("git clone https://github.com/2ndQuadrant/pg-tpch")
os.chdir("pg-tpch")
os.chdir("dbgen")
os.system("make")
Scalefactor=raw_input("Please type the scale factor number you would like for tpch : ")
os.system("./dbgen -s %s" %Scalefactor)
os.system("for i in `ls *.tbl`; do sed 's/|$//' $i > ${i/tbl/csv}; echo $i; done;")
        
Dirvar=raw_input("Please type the directory name in which you have Postgres installed or you want to install it: ")

pathDirvar=os.path.join(home,Dirvar)
pathupdatecommit=os.path.join(home,Dirvar,'postgres')

if os.path.exists(os.path.join(home,Dirvar))==True:
    print "PostGresql is already installed."
    os.chdir(home)
    os.chdir(Dirvar)
    os.chdir('postgres')
    Pversion=os.popen('psql -V').read()
    print ("The version of Postgresql you have installed is : %s " %Pversion)
    Datadir=raw_input("Please identify the directory where the data for this database will reside: ")
    while os.path.exists(os.path.join(home,Dirvar,Datadir))==True:
        Datadir=raw_input("The directory already exists, please specify the directory where the data for this database will reside: ")
    else:
        os.chdir(pathDirvar)
        os.chdir('bin')
	os.system("./initdb -D %s" % Datadir)
	print "Starting PostgreSQL.."
        os.system("./pg_ctl -D %s -l logfile start" % Datadir)
        DBname=raw_input("Please specify the name of your database: ")
        os.system("./createdb %s" % DBname)
        print ("Creating the tables...")

        conn=psycopg2.connect("dbname='%s' user='%s' host='localhost'" % (DBname,Uservar))
 
        cur=conn.cursor()

        cur.execute("""Create table nation (n_nationkey integer, n_name text, n_regionkey integer, n_comment text)""")

        cur.execute("""Create table supplier (s_suppkey integer, s_name text, s_address text, s_nationkey integer, s_phone text, s_acctbal decimal, s_comment text)""")

        cur.execute("""Create table region (r_regionkey integer, r_name text, r_comment text)""")

        cur.execute( """Create table customer (c_custkey integer,c_name text, c_address text, c_nationkey integer, c_phone text, c_acctbal decimal, c_mktsegment text, c_comment text)""")

        cur.execute("""Create table orders (o_orderkey integer, o_custkey integer, o_orderstatus text,o_totalprice decimal, o_orderdate date, o_orderpriority text, o_shippriority text, o_comment text, extracol text)""")

        cur.execute(""" Create table part(p_partkey integer, p_name text, p_mfgr text, p_brand text, p_type text, p_size integer, p_container text, p_retailprice decimal, p_comment text)""")

        cur.execute("""Create table partsupp (ps_partkey integer, ps_suppkey integer, ps_availqty integer, ps_supplycost decimal, ps_comment text)""")

        cur.execute("""Create table lineitem (l_orderkey integer, l_partkey integer, l_suppkey integer, l_linenumber integer, l_quantity real, l_extendedprice decimal, l_discount decimal, l_tax decimal,l_returnflag text, l_linestatus text, l_shipdate date, l_commitdate date, l_receiptdate date, l_shipinstruct text, l_shipmode text, l_comment text)""")
        startloading1=time.time()
        cur.execute("""Copy customer from '%s' (format csv, delimiter ('|'));""" % pathcustomer)

        cur.execute("""Copy lineitem from '%s' (format csv, delimiter ('|'));""" % pathlineitem)

        cur.execute("""Copy nation from '%s' (format csv, delimiter ('|'));""" % pathnation)

        cur.execute("""Copy orders from '%s' (format csv, delimiter ('|'));""" % pathorders)

        cur.execute("""Copy part  from '%s' (format csv, delimiter ('|'));""" % pathpart)

        cur.execute("""Copy partsupp from '%s' (format csv, delimiter ('|'));""" % pathpartsupp)

        cur.execute("""Copy region from '%s' (format csv, delimiter ('|'));""" % pathregion)

        cur.execute("""Copy supplier from '%s' (format csv, delimiter ('|'));""" % pathsupplier)
        endloading1=time.time()
        difloading1=endloading1-startloading1

        print "Done loading the tables"
        print ("Time loading the tables : %s seconds" %difloading1)
        num_times=6
        run_times1=[]
        for i in range(1,num_times):
            start1=time.time()
            cur.execute("""Select l_returnflag,l_linestatus,sum(l_quantity) as sum_qty, sum(l_extendedprice) as sum_base_price,sum(l_extendedprice * (1 - l_discount)) as sum_disc_price,sum(l_extendedprice * (1-l_discount) * (1 + l_tax)) as sum_charge, avg(l_quantity) as avg_qty,avg(l_extendedprice) as avg_price, avg(l_discount) as avg_disc,count(*) as count_order from lineitem where l_shipdate <= date '1998-12-01' - interval '90' day group by l_returnflag,l_linestatus order by l_returnflag,l_linestatus;""")
            end1=time.time()
            q1=end1-start1
            run_times1.append(q1)
            print ("Elapsed time for the %d run of the 1st query is : %s seconds" %(i,q1))
    
        q1_cold=run_times1[0]
        q1_min_hot=min(run_times1[1:])
        q1_max_hot=max(run_times1[1:])
        q1_avg=(sum(run_times1[1:]))/(len(run_times1)-1)
        print ("The cold run for the 1st query was: %s seconds" %q1_cold)
        print ("The fastest time for the 1st query hot runs was : %s seconds" %q1_min_hot)
        print ("The slowest time for the hot runs was : %s" %q1_max_hot)
        print ("Average time for the 1st query hot runs is : %s seconds" %q1_avg)
        print ("---------------------------------------------------------------")


        run_times2=[]
        for i in range(1,num_times):
            start2=time.time()
            cur.execute("""Select s_acctbal,s_name,n_name,p_partkey,p_mfgr,s_address,s_phone,s_comment from part,supplier,partsupp,nation,region where p_partkey=ps_partkey and s_suppkey=ps_suppkey and p_size=15 and p_type like '%BRASS' and s_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='EUROPE' and ps_supplycost=(Select min(ps_supplycost) from partsupp,supplier,nation,region where p_partkey=ps_partkey and s_suppkey=ps_suppkey and s_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='EUROPE') order by s_acctbal desc, n_name, s_name, p_partkey limit 100;""")
            end2=time.time()
            q2=end2-start2
            run_times2.append(q2)
            print ("Elapsed time for the %d run of the 2nd query is : %s seconds" %(i,q2))

        
        q2_cold=run_times2[0]
        q2_min_hot=min(run_times2[1:])
        q2_max_hot=max(run_times2[1:])
        q2_avg=(sum(run_times2[1:]))/(len(run_times2)-1)
        print ("The cold run for the 2nd query was: %s seconds" %q2_cold)
        print ("The fastest time for the 2nd query hot runs was : %s seconds" %q2_min_hot)
        print ("The slowest time for 2nd query hot runs was : %s" %q2_max_hot)
        print ("Average time for the 2nd query hot runs is : %s seconds" %q2_avg)
        print ("---------------------------------------------------------------")
        run_times3=[]
        for i in range(1,num_times):
            start3=time.time()
            cur.execute("""Select l_orderkey,sum(l_extendedprice * (1-l_discount)) as revenue, o_orderdate,o_shippriority from customer,orders,lineitem where c_mktsegment='BUILDING' and c_custkey=o_custkey and l_orderkey=o_orderkey and o_orderdate<date '1995-03-15' and l_shipdate>date '1995-03-15' group by l_orderkey, o_orderdate, o_shippriority order by revenue desc, o_orderdate limit 10;""")
            end3=time.time()
            q3=end3-start3
            run_times3.append(q3)
            print ("Elapsed time for the %d run of the 3rd query is : %s seconds" %(i, q3))

        q3_cold=run_times3[0]
        q3_min_hot=min(run_times3[1:])
        q3_max_hot=max(run_times3[1:])
        q3_avg=(sum(run_times3[1:]))/(len(run_times3)-1)
        print ("The cold run for the 3rd query was: %s seconds" %q3_cold)
        print ("The fastest time for the 3rd query hot runs was : %s seconds" %q3_min_hot)
        print ("The slowest time for the 3rd query hot runs was : %s" %q3_max_hot)
        print ("Average time for the 3rd query hot runs is : %s seconds" %q3_avg)
        print ("---------------------------------------------------------------")

        run_times4=[]
        for i in range(1,num_times):
            start4=time.time()
            cur.execute("""Select o_orderpriority, count(*) as order_count from orders where o_orderdate>= date '1993-07-01' and o_orderdate < date '1993-07-01' + interval '3' month and exists ( Select * from lineitem where l_orderkey=o_orderkey and l_commitdate < l_receiptdate) group by o_orderpriority order by o_orderpriority;""")
            end4=time.time()
            q4=end4-start4
            run_times4.append(q4)
            print ("Elapsed time for the %d run of the 4th query is : %s seconds" % (i,q4))

        q4_cold=run_times4[0]
        q4_min_hot=min(run_times4[1:])
        q4_max_hot=max(run_times4[1:])
        q4_avg=(sum(run_times4[1:]))/(len(run_times4)-1)
        print ("The cold run for the 4th query was: %s seconds" %q4_cold)
        print ("The fastest time for the 4th query hot runs was : %s seconds" %q4_min_hot)
        print ("The slowest time for the 4th query runs was : %s" %q4_max_hot)
        print ("Average time for the 4th query hot runs is : %s seconds" %q4_avg)
        print ("---------------------------------------------------------------")
        run_times5=[]
        for i in range(1, num_times):
            start5 = time.time()
            cur.execute("""Select n_name, sum(l_extendedprice * (1-l_discount)) as revenue from customer, orders, lineitem, supplier, nation, region where c_custkey=o_custkey and l_orderkey=o_orderkey and l_suppkey=s_suppkey and c_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='ASIA' and o_orderdate >=date '1994-01-01' and o_orderdate>=date '1994-01-01' and o_orderdate<date '1994-01-01' + interval '1' year group by n_name order by revenue desc;""")
            end5=time.time()
            q5 = end5 - start5
            run_times5.append(q5)
            print ("Elapsed time for the run %d of the 5th query is : %s seconds" % (i, q5) )
        
        q5_cold=run_times5[0]
        q5_min_hot=min(run_times5[1:])
        q5_max_hot=max(run_times5[1:])
        q5_avg=(sum(run_times5[1:]))/(len(run_times5)-1)
        print ("The cold run for the 5th query was: %s seconds" %q5_cold)
        print ("The fastest time for the 5th query hot runs was : %s seconds" %q5_min_hot)
        print ("The slowest time for the 5th hot runs was : %s" %q5_max_hot)
        print ("Average time for the 5th query hot runs is : %s seconds" %q5_avg)
        print ("---------------------------------------------------------------")
 

        run_times6=[]
        for i in range(1,num_times):
            start6=time.time()
            cur.execute("""Select sum(l_extendedprice * l_discount) as revenue from lineitem where l_shipdate>=date '1994-01-01' and l_shipdate < date '1994-01-01' + interval '1' year and l_discount between 0.06 - 0.01 and 0.06 + 0.01 and l_quantity<24;""")
            end6=time.time()
            q6=end6-start6
            run_times6.append(q6)
            print ("Elapsed time for the %d run of the 6th query is : %s seconds" % (i,q6))

        q6_cold=run_times6[0]
        q6_min_hot=min(run_times6[1:])
        q6_max_hot=max(run_times6[1:])
        q6_avg=(sum(run_times6[1:]))/(len(run_times6)-1)
        print ("The cold run for the 6th query was: %s seconds" %q6_cold)
        print ("The fastest time for the 6th query hot runs was : %s seconds" %q6_min_hot)
        print ("The slowest time for the 6th hot runs was : %s" %q6_max_hot)
        print ("Average time for the 6th query hot runs is : %s seconds" %q6_avg)
        print ("---------------------------------------------------------------")

        run_times7=[]
        for i in range(1,num_times):
            start7=time.time()
            cur.execute("""Select supp_nation,cust_nation,l_year,sum(volume) as revenue from ( Select n1.n_name as supp_nation, n2.n_name as cust_nation, extract(year from l_shipdate) as l_year, l_extendedprice * (1-l_discount) as volume from supplier,lineitem,orders,customer,nation n1, nation n2 where s_suppkey=l_suppkey and o_orderkey=l_orderkey and c_custkey=o_custkey and s_nationkey=n1.n_nationkey and c_nationkey=n2.n_nationkey and ((n1.n_name='FRANCE' and n2.n_name='GERMANY') or (n1.n_name='GERMANY' and n2.n_name='FRANCE')) and l_shipdate between date '1995-01-01' and date '1996-12-31') as shipping group by supp_nation,cust_nation,l_year order by supp_nation,cust_nation,l_year;""")
            end7=time.time()
            q7=end7-start7
            run_times7.append(q7)
            print ("Elapsed time for the %d run of the 7th query is : %s seconds" %(i, q7))

        q7_cold=run_times7[0]
        q7_min_hot=min(run_times7[1:])
        q7_max_hot=max(run_times7[1:])
        q7_avg=(sum(run_times7[1:]))/(len(run_times7)-1)
        print ("The cold run for the 7th query was: %s seconds" %q7_cold)
        print ("The fastest time for the 7th query hot runs was : %s seconds" %q7_min_hot)
        print ("The slowest time for the 7th hot runs was : %s" %q7_max_hot)
        print ("Average time for the 7th query hot runs is : %s seconds" %q7_avg)
        print ("---------------------------------------------------------------")

        run_times8=[]
        for i in range(1,num_times):
            start8=time.time()
            cur.execute("""Select o_year, sum(case when nation='BRAZIL' then volume else 0 end)/sum(volume) as mkt_share from (Select extract(year from o_orderdate) as o_year,l_extendedprice * (1-l_discount) as volume, n2.n_name as nation from part, supplier, lineitem, orders, customer, nation n1, nation n2, region where p_partkey=l_partkey and s_suppkey=l_suppkey and l_orderkey=o_orderkey and o_custkey=c_custkey and c_nationkey=n1.n_nationkey and n1.n_regionkey=r_regionkey and r_name='AMERICA' and s_nationkey=n2.n_nationkey and o_orderdate between date '1995-01-01' and date '1996-12-31' and p_type='ECONOMY ANODIZED STEEL') as all_nations group by o_year order by o_year;""")
            end8=time.time()
            q8=end8-start8
            run_times8.append(q8)
            print ("Elapsed time for the %d run of the 8th query is : %s seconds" %(i,q8))

        q8_cold=run_times8[0]
        q8_min_hot=min(run_times8[1:])
        q8_max_hot=max(run_times8[1:])
        q8_avg=(sum(run_times8[1:]))/(len(run_times8)-1)
        print ("The cold run for the 8th query was: %s seconds" %q8_cold)
        print ("The fastest time for the 8th query hot runs was : %s seconds" %q8_min_hot)
        print ("The slowest time for the 8th hot runs was : %s" %q8_max_hot)
        print ("Average time for the 8th query hot runs is : %s seconds" %q8_avg)
        print ("---------------------------------------------------------------")

        run_times9=[]
        for i in range(1,num_times):
            start9=time.time()
            cur.execute("""Select nation,o_year,sum(amount) as sum_profit from ( Select n_name as nation, extract(year from o_orderdate) as o_year, l_extendedprice * (1-l_discount)-ps_supplycost * l_quantity as amount from part, supplier, lineitem, partsupp, orders, nation where s_suppkey=l_suppkey and ps_suppkey=l_suppkey and ps_partkey=l_partkey and p_partkey=l_partkey and o_orderkey=l_orderkey and s_nationkey=n_nationkey and p_name like '%green%') as profit group by nation, o_year order by nation,o_year desc;""")
            end9=time.time()
            q9=end9-start9
            run_times9.append(q9)
            print ("Elapsed time for the %d run of the 9th query is : %s seconds" % (i,q9))

        q9_cold=run_times9[0]
        q9_min_hot=min(run_times9[1:])
        q9_max_hot=max(run_times9[1:])
        q9_avg=(sum(run_times9[1:]))/(len(run_times9)-1)
        print ("The cold run for the 9th query was: %s seconds" %q9_cold)
        print ("The fastest time for the 9th query hot runs was : %s seconds" %q9_min_hot)
        print ("The slowest time for the 9th hot runs was : %s" %q9_max_hot)
        print ("Average time for the 9th query hot runs is : %s seconds" %q9_avg)
        print ("---------------------------------------------------------------")
        
        run_times10=[]
        for i in range(1,num_times):
            start10=time.time()
            cur.execute("""Select c_custkey, c_name, sum(l_extendedprice * (1-l_discount)) as revenue, c_acctbal, n_name, c_address, c_phone, c_comment from customer, orders, lineitem, nation where c_custkey=o_custkey and l_orderkey=o_orderkey and o_orderdate>=date '1993-10-01' and o_orderdate < date '1993-10-01' + interval '3' month and l_returnflag='R' and c_nationkey=n_nationkey group by c_custkey, c_name, c_acctbal, c_phone, n_name, c_address, c_comment order by revenue desc limit 20;""")
            end10=time.time()
            q10=end10-start10
            run_times10.append(q10)
            print ("Elapsed time for the %d run of the 10th query is : %s seconds" %(i,q10))

        q10_cold=run_times10[0]
        q10_min_hot=min(run_times10[1:])
        q10_max_hot=max(run_times10[1:])
        q10_avg=(sum(run_times10[1:]))/(len(run_times10)-1)
        print ("The cold run for the 10th query was: %s seconds" %q10_cold)
        print ("The fastest time for the 10th query hot runs was : %s seconds" %q10_min_hot)
        print ("The slowest time for the 10th hot runs was : %s" %q10_max_hot)
        print ("Average time for the 10th query hot runs is : %s seconds" %q10_avg)
        print ("---------------------------------------------------------------")

        run_times11=[]
        for i in range(1,num_times):
            start11=time.time()
            cur.execute("""Select ps_partkey, sum(ps_supplycost * ps_availqty) as value from partsupp, supplier, nation where ps_suppkey=s_suppkey and s_nationkey=n_nationkey and n_name='GERMANY' group by ps_partkey having sum(ps_supplycost * ps_availqty)>(Select sum(ps_supplycost * ps_availqty) * 0.0001000000 from partsupp, supplier, nation where ps_suppkey=s_suppkey and s_nationkey=n_nationkey and n_name='GERMANY') order by value desc;""")
            end11=time.time()
            q11=end11-start11
            run_times11.append(q11)
            print ("Elapsed time for the %d run of the 11th query is : %s seconds" %(i, q11))

        q11_cold=run_times11[0]
        q11_min_hot=min(run_times11[1:])
        q11_max_hot=max(run_times11[1:])
        q11_avg=(sum(run_times11[1:]))/(len(run_times11)-1)
        print ("The cold run for the 11th query was: %s seconds" %q11_cold)
        print ("The fastest time for the 11th query hot runs was : %s seconds" %q11_min_hot)
        print ("The slowest time for the 11th hot runs was : %s" %q11_max_hot)
        print ("Average time for the 11th query hot runs is : %s seconds" %q11_avg)
        print ("---------------------------------------------------------------")
        run_times12=[]
        for i in range(1,num_times):
            start12=time.time()
            cur.execute("""Select l_shipmode,sum(case when o_orderpriority='1-URGENT' or o_orderpriority='2-HIGH' then 1 else 0 end) as high_line_count, sum(case when o_orderpriority <> '1=URGENT' and o_orderpriority <> '2-HIGH' then 1 else 0 end) as low_line_count from orders,lineitem where o_orderkey=l_orderkey and l_shipmode in('MAIL','SHIP') and l_commitdate < l_receiptdate and l_shipdate < l_commitdate and l_receiptdate >= date '1994-01-01' and l_receiptdate < date '1004-01-01'+ interval '1' year group by l_shipmode order by l_shipmode;""")
            end12=time.time()
            q12=end12-start12
            run_times12.append(q12)
            print ("Elapsed time for the %d run of the 12th query is : %s seconds" %(i,q12))

        q12_cold=run_times12[0]
        q12_min_hot=min(run_times12[1:])
        q12_max_hot=max(run_times12[1:])
        q12_avg=(sum(run_times12[1:]))/(len(run_times12)-1)
        print ("The cold run for the 12th query was: %s seconds" %q12_cold)
        print ("The fastest time for the 12th query hot runs was : %s seconds" %q12_min_hot)
        print ("The slowest time for the 12th hot runs was : %s" %q12_max_hot)
        print ("Average time for the 12th query hot runs is : %s seconds" %q12_avg)
        print ("---------------------------------------------------------------")

        run_times13=[]
        for i in range(1,num_times):
            start13=time.time()
            cur.execute("""Select c_count,count(*) as custdist from ( Select c_custkey, count(o_orderkey) from customer left outer join orders on c_custkey=o_custkey and o_comment not like '%special%requests%' group by c_custkey) as c_orders(c_custkey,c_count) group by c_count order by custdist desc, c_count desc;""")
            end13=time.time()
            q13=end13-start13
            run_times13.append(q13)
            print ("Elapsed time for the %d run of the 13th query is %s seconds" %(i,q13))

        q13_cold=run_times13[0]
        q13_min_hot=min(run_times13[1:])
        q13_max_hot=max(run_times13[1:])
        q13_avg=(sum(run_times13[1:]))/(len(run_times13)-1)
        print ("The cold run for the 13th query was: %s seconds" %q13_cold)
        print ("The fastest time for the 13th query hot runs was : %s seconds" %q13_min_hot)
        print ("The slowest time for the 13th hot runs was : %s" %q13_max_hot)
        print ("Average time for the 13th query hot runs is : %s seconds" %q13_avg)
        print ("---------------------------------------------------------------")
        run_times14=[]
        for i in range(1,num_times):
            start14=time.time()
            cur.execute("""Select 100.00 * sum(case when p_type like 'PROMO%' then l_extendedprice * (1-l_discount) else 0 end) / sum(l_extendedprice * (1-l_discount)) as promo_revenue from lineitem,part where l_partkey=p_partkey and l_shipdate >= date '1995-09-01' and l_shipdate < date '1995-09-01' + interval '1' month;""")
            end14=time.time()
            q14=end14-start14
            run_times14.append(q14)
            print ("Elapsed time for the %d run of the 14th query is %s seconds" %(i,q14))

        q14_cold=run_times14[0]
        q14_min_hot=min(run_times14[1:])
        q14_max_hot=max(run_times14[1:])
        q14_avg=(sum(run_times14[1:]))/(len(run_times14)-1)
        print ("The cold run for the 14th query was: %s seconds" %q14_cold)
        print ("The fastest time for the 14th query hot runs was : %s seconds" %q14_min_hot)
        print ("The slowest time for the 14th hot runs was : %s" %q14_max_hot)
        print ("Average time for the 14th query hot runs is : %s seconds" %q14_avg)
        print ("---------------------------------------------------------------")
    
        run_times15=[]
        for i in range(1,num_times):
            start15=time.time()
            cur.execute("""Create view revenue0(supplier_no, total_revenue) as select l_suppkey, sum(l_extendedprice * (1-l_discount)) from lineitem where l_shipdate >= date '1996-01-01' and l_shipdate < date '1996-01-01' + interval '3' month group by l_suppkey; Select s_suppkey,s_name, s_address, s_phone, total_revenue from supplier, revenue0 where s_suppkey=supplier_no and total_revenue=(select max(total_revenue) from revenue0) order by s_suppkey; drop view revenue0;""")
            end15=time.time()
            q15=end15-start15
            run_times15.append(q15)
            print ("Elapsed time for the %d run of the 15th query is %s seconds" %(i,q15))

        q15_cold=run_times15[0]
        q15_min_hot=min(run_times15[1:])
        q15_max_hot=max(run_times15[1:])
        q15_avg=(sum(run_times15[1:]))/(len(run_times15)-1)
        print ("The cold run for the 15th query was: %s seconds" %q15_cold)
        print ("The fastest time for the 15th query hot runs was : %s seconds" %q15_min_hot)
        print ("The slowest time for the 15th hot runs was : %s" %q15_max_hot)
        print ("Average time for the 15th query hot runs is : %s seconds" %q15_avg)
        print ("---------------------------------------------------------------")

        run_times16=[]
        for i in range(1,num_times):
            start16=time.time()
            cur.execute("""Select p_brand, p_type, p_size, count(distinct ps_suppkey) as supplier_cnt from partsupp, part where p_partkey=ps_partkey and p_brand <> 'Brand#45'and p_type not like 'MEDIUM POLISHED%' and p_size in (49,14,23,45,19,3,36,9) and ps_suppkey not in( Select s_suppkey from supplier where s_comment like '%Customer%Complaints%') group by p_brand,p_type,p_size order by supplier_cnt desc, p_brand,p_type, p_size;""")
            end16=time.time()
            q16=end16-start16
            run_times16.append(q16)
            print ("Elapsed time for the %d run of the 16th query is %s seconds" %(i,q16))

        q16_cold=run_times16[0]
        q16_min_hot=min(run_times16[1:])
        q16_max_hot=max(run_times16[1:])
        q16_avg=(sum(run_times16[1:]))/(len(run_times16)-1)
        print ("The cold run for the 16th query was: %s seconds" %q16_cold)
        print ("The fastest time for the 16th query hot runs was : %s seconds" %q16_min_hot)
        print ("The slowest time for the 16th hot runs was : %s" %q16_max_hot)
        print ("Average time for the 16th query hot runs is : %s seconds" %q16_avg)
        print ("---------------------------------------------------------------")

        run_times17=[]
        for i in range(1,num_times):
            start17=time.time()
            cur.execute("""Select sum(l_extendedprice) / 7.0 as avg_yearly from lineitem, part where p_partkey=l_partkey and p_brand='Brand#23' and p_container='MED BOX' and l_quantity < (Select 0.2 * avg(l_quantity) from lineitem where l_partkey=p_partkey);""")
            end17=time.time()
            q17=end17-start17
            run_times17.append(q17)
            print ("Elapsed time for the %d run of the 17th query is %s seconds" %(i,q17))

        q17_cold=run_times17[0]
        q17_min_hot=min(run_times17[1:])
        q17_max_hot=max(run_times17[1:])
        q17_avg=(sum(run_times17[1:]))/(len(run_times17)-1)
        print ("The cold run for the 17th query was: %s seconds" %q17_cold)
        print ("The fastest time for the 17th query hot runs was : %s seconds" %q17_min_hot)
        print ("The slowest time for the 17th hot runs was : %s" %q17_max_hot)
        print ("Average time for the 17th query hot runs is : %s seconds" %q17_avg)
        print ("---------------------------------------------------------------")

        run_times18=[]
        for i in range(1,num_times):
            start18=time.time()
            cur.execute("""Select c_name,c_custkey,o_orderkey, o_orderdate, o_totalprice, sum(l_quantity) from customer, orders, lineitem where o_orderkey in (Select l_orderkey from lineitem group by l_orderkey having sum(l_quantity)>300) and c_custkey=o_custkey and o_orderkey=l_orderkey group by c_name, c_custkey, o_orderkey,o_orderdate,o_totalprice order by o_totalprice desc, o_orderdate limit 100;""")
            end18=time.time()
            q18=end18-start18
            run_times18.append(q18)
            print ("Elapsed time for the %d run of the 18th query is %s seconds" %(i,q18))

        q18_cold=run_times18[0]
        q18_min_hot=min(run_times18[1:])
        q18_max_hot=max(run_times18[1:])
        q18_avg=(sum(run_times18[1:]))/(len(run_times18)-1)
        print ("The cold run for the 18th query was: %s seconds" %q18_cold)
        print ("The fastest time for the 18th query hot runs was : %s seconds" %q18_min_hot)
        print ("The slowest time for the 18th hot runs was : %s" %q18_max_hot)
        print ("Average time for the 18th query hot runs is : %s seconds" %q18_avg)
        print ("---------------------------------------------------------------")

        run_times19=[]
        for i in range(1,num_times):
            start19=time.time()
            cur.execute("""Select sum(l_extendedprice * (1-l_discount)) as revenrue from lineitem, part where ( p_partkey=l_partkey and p_brand='Brand#12' and p_container in ('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG') and l_quantity >=1 and l_quantity <= 1+10 and p_size between 1 and 5 and l_shipmode in ('AIR', 'AIR REG') and l_shipinstruct='DELIVER IN PERSON') or ( p_partkey=l_partkey and p_brand ='Brand#23' and p_container in ('MED BAG','MED BOX', 'MED PKG', 'MED PACK') and l_quantity >=10 and l_quantity <= 10+10 and p_size between 1 and 10 and l_shipmode in ('AIR','AIR REG') and l_shipinstruct='DELIVER IN PERSON') or (p_partkey=l_partkey and p_brand ='Brand#34' and p_container in ('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG') and l_quantity >= 20 and l_quantity <=20+10 and p_size between 1 and 15 and l_shipmode in ('AIR', 'AIR REG') and l_shipinstruct='DELIVER IN PERSON');""")
            end19=time.time()
            q19=end19-start19
            run_times19.append(q19)
            print ("Elapsed time for the %d run of the 19th query is %s seconds" %(i,q19))

        q19_cold=run_times19[0]
        q19_min_hot=min(run_times19[1:])
        q19_max_hot=max(run_times19[1:])
        q19_avg=(sum(run_times19[1:]))/(len(run_times19)-1)
        print ("The cold run for the 19th query was: %s seconds" %q19_cold)
        print ("The fastest time for the 19th query hot runs was : %s seconds" %q19_min_hot)
        print ("The slowest time for the 19th hot runs was : %s" %q19_max_hot)
        print ("Average time for the 19th query hot runs is : %s seconds" %q19_avg)
        print ("---------------------------------------------------------------")

        run_times20=[]
        for i in range(1,num_times):
            start20=time.time()
            cur.execute("""Select s_name, s_address from supplier, nation where s_suppkey in ( Select ps_suppkey from partsupp where ps_partkey in (Select p_partkey from part where p_name like 'forest%' ) and ps_availqty > ( Select 0.5 * sum(l_quantity) from lineitem where l_partkey=ps_partkey and l_suppkey = ps_suppkey and l_shipdate >= date '1994-01-01' and l_shipdate < date '1994-01-01' + interval '1' year)) and s_nationkey=n_nationkey and s_name ='CANADA' order by s_name;""")
            end20=time.time()
            q20=end20-start20
            run_times20.append(q20)
            print ("Elapsed time for the %d run of the 20th query is %s seconds" %(i,q20))

        q20_cold=run_times20[0]
        q20_min_hot=min(run_times20[1:])
        q20_max_hot=max(run_times20[1:])
        q20_avg=(sum(run_times20[1:]))/(len(run_times20)-1)
        print ("The cold run for the 20th query was: %s seconds" %q20_cold)
        print ("The fastest time for the 20th query hot runs was : %s seconds" %q20_min_hot)
        print ("The slowest time for the 20th hot runs was : %s" %q20_max_hot)
        print ("Average time for the 20th query hot runs is : %s seconds" %q20_avg)
        print ("---------------------------------------------------------------")
        run_times21=[]
        for i in range(1, num_times):
            start21=time.time()
            cur.execute("""Select s_name, count(*) as numwait from supplier, lineitem l1, orders, nation where s_suppkey=l1.l_suppkey and o_orderkey=l1.l_orderkey and o_orderstatus='F' and l1.l_receiptdate > l1.l_commitdate and exists ( Select * from lineitem l2 where l2.l_orderkey=l1.l_orderkey and l2.l_suppkey <> l1.l_suppkey) and not exists ( Select * from lineitem l3 where l3.l_orderkey =l1.l_orderkey and l3.l_suppkey<> l1.l_suppkey and l3.l_receiptdate > l3.l_commitdate) and s_nationkey=n_nationkey and n_name='SAUDI ARABIA' group by s_name order by numwait desc, s_name limit 100;""")
            end21=time.time()
            q21=end21-start21
            run_times21.append(q21)
            print ("Elapsed time for the %d run of the 21st query is %s seconds" %(i,q21))

        q21_cold=run_times21[0]
        q21_min_hot=min(run_times21[1:])
        q21_max_hot=max(run_times21[1:])
        q21_avg=(sum(run_times21[1:]))/(len(run_times21)-1)
        print ("The cold run for the 21st query was: %s seconds" %q21_cold)
        print ("The fastest time for the 21st query hot runs was : %s seconds" %q21_min_hot)
        print ("The slowest time for the 21st hot runs was : %s" %q21_max_hot)
        print ("Average time for the 21st query hot runs is : %s seconds" %q21_avg)
        print ("---------------------------------------------------------------")

        run_times22=[]
        for i in range(1, num_times):
            start22=time.time()
            cur.execute("""Select cntrycode, count(*) as numcust, sum(c_acctbal) as totacctbal from ( Select substring(c_phone from 1 for 2) as cntrycode,c_acctbal from customer where substring(c_phone from 1 for 2) in ('13','31','23','29','30','18','17') and c_acctbal > ( Select avg(c_acctbal) from customer where c_acctbal >0.00 and substring (c_phone from 1 for 2) in ('13','31','23','29','30','18','17')) and not exists ( Select * from orders where o_custkey=c_custkey)) as custsale group by cntrycode order by cntrycode;""")
            end22=time.time()
            q22=end22-start22
            run_times22.append(q22)
            print ("Elapsed time for the %d run of the 22nd query is %s seconds" %(i,q22))

        q22_cold=run_times22[0]
        q22_min_hot=min(run_times22[1:])
        q22_max_hot=max(run_times22[1:])
        q22_avg=(sum(run_times22[1:]))/(len(run_times22)-1)
        print ("The cold run for the 22nd query was: %s seconds" %q22_cold)
        print ("The fastest time for the 22nd query hot runs was : %s seconds" %q22_min_hot)
        print ("The slowest time for the 22nd hot runs was : %s" %q22_max_hot)
        print ("Average time for the 22nd query hot runs is : %s seconds" %q22_avg)
        print ("---------------------------------------------------------------")

#---------------------UPDATE TRY---------------------------------

    os.chdir(pathupdatecommit)
    print "Current commit version is : "
    ComValue1=os.system("git rev-parse HEAD")
    print "Updating to the new commit of Postgresql.."
    os.system("git pull https://github.com/postgres/postgres")
    print "--------------------------------------------------"
    print "Current commit version is : "
    ComValue2=os.system("git rev-parse HEAD")
    #while ComValue1==ComValue2:
        #print ("There is no new commit available, waiting for ? mins..")
        #os.system("sleep 3m")
        #os.system("git pull https://github.com/postgres/postgres")
        #ComValue2=os.system("git rev-parse")
    pathpostupdate=os.path.join(home,Dirvar,'bin')
    os.chdir(pathpostupdate)
    print "Dropping the tables and reloading them..."
    cur.execute("""Drop table nation;""")
    cur.execute("""Drop table supplier;""")
    cur.execute("""Drop table region;""")
    cur.execute("""Drop table customer;""")
    cur.execute("""Drop table orders;""")
    cur.execute("""Drop table part;""")
    cur.execute("""Drop table partsupp;""")
    cur.execute("""Drop table lineitem;""")
    
    cur.execute("""Create table nation (n_nationkey integer, n_name text, n_regionkey integer, n_comment text)""")

    cur.execute("""Create table supplier (s_suppkey integer, s_name text, s_address text, s_nationkey integer, s_phone text, s_acctbal decimal, s_comment text)""")

    cur.execute("""Create table region (r_regionkey integer, r_name text, r_comment text)""")

    cur.execute( """Create table customer (c_custkey integer,c_name text, c_address text, c_nationkey integer, c_phone text, c_acctbal decimal, c_mktsegment text, c_comment text)""")

    cur.execute("""Create table orders (o_orderkey integer, o_custkey integer, o_orderstatus text,o_totalprice decimal, o_orderdate date, o_orderpriority text, o_shippriority text, o_comment text, extracol text)""")

    cur.execute(""" Create table part(p_partkey integer, p_name text, p_mfgr text, p_brand text, p_type text, p_size integer, p_container text, p_retailprice decimal, p_comment text)""")

    cur.execute("""Create table partsupp (ps_partkey integer, ps_suppkey integer, ps_availqty integer, ps_supplycost decimal, ps_comment text)""")

    cur.execute("""Create table lineitem (l_orderkey integer, l_partkey integer, l_suppkey integer, l_linenumber integer, l_quantity real, l_extendedprice decimal, l_discount decimal, l_tax decimal,l_returnflag text, l_linestatus text, l_shipdate date, l_commitdate date, l_receiptdate date, l_shipinstruct text, l_shipmode text, l_comment text)""")
    startloading2=time.time()
    cur.execute("""Copy customer from '%s' (format csv, delimiter ('|'));""" % pathcustomer)

    cur.execute("""Copy lineitem from '%s' (format csv, delimiter ('|'));""" % pathlineitem)

    cur.execute("""Copy nation from '%s' (format csv, delimiter ('|'));""" % pathnation)

    cur.execute("""Copy orders from '%s' (format csv, delimiter ('|'));""" % pathorders)

    cur.execute("""Copy part  from '%s' (format csv, delimiter ('|'));""" % pathpart)

    cur.execute("""Copy partsupp from '%s' (format csv, delimiter ('|'));""" % pathpartsupp)

    cur.execute("""Copy region from '%s' (format csv, delimiter ('|'));""" % pathregion)

    cur.execute("""Copy supplier from '%s' (format csv, delimiter ('|'));""" % pathsupplier)
    endloading2=time.time()
    difloading2=endloading2-startloading2
    totaldifloading=difloading1-difloading2
    print "Done loading the tables"
    print ("Time loading the tables with the new commit: %s seconds" %difloading2)
    print ("Difference in loading the tables from the previous version is : %s seconds" %totaldifloading)
    

    num_times=6
    run_times1b=[]
    for i in range(1,num_times):
            start1b=time.time()
            cur.execute("""Select l_returnflag,l_linestatus,sum(l_quantity) as sum_qty, sum(l_extendedprice) as sum_base_price,sum(l_extendedprice * (1 - l_discount)) as sum_disc_price,sum(l_extendedprice * (1-l_discount) * (1 + l_tax)) as sum_charge, avg(l_quantity) as avg_qty,avg(l_extendedprice) as avg_price, avg(l_discount) as avg_disc,count(*) as count_order from lineitem where l_shipdate <= date '1998-12-01' - interval '90' day group by l_returnflag,l_linestatus order by l_returnflag,l_linestatus;""")
            end1b=time.time()
            q1b=end1b-start1b
            run_times1b.append(q1b)
            print ("Elapsed time for the %d run of the 1st query is : %s seconds" %(i,q1b))
    
    q1_coldb=run_times1b[0]
    q1_min_hotb=min(run_times1b[1:])
    q1_max_hotb=max(run_times1b[1:])
    q1_avgb=(sum(run_times1b[1:]))/(len(run_times1b)-1)
    print ("The cold run for the 1st query was: %s seconds" %q1_coldb)
    print ("The fastest time for the 1st query hot runs was : %s seconds" %q1_min_hotb)
    print ("The slowest time for the hot runs was : %s" %q1_max_hotb)
    print ("Average time for the 1st query hot runs is : %s seconds" %q1_avgb)
    print ("---------------------------------------------------------------")

    run_times2b=[]
    for i in range(1,num_times):
            start2b=time.time()
            cur.execute("""Select s_acctbal,s_name,n_name,p_partkey,p_mfgr,s_address,s_phone,s_comment from part,supplier,partsupp,nation,region where p_partkey=ps_partkey and s_suppkey=ps_suppkey and p_size=15 and p_type like '%BRASS' and s_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='EUROPE' and ps_supplycost=(Select min(ps_supplycost) from partsupp,supplier,nation,region where p_partkey=ps_partkey and s_suppkey=ps_suppkey and s_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='EUROPE') order by s_acctbal desc, n_name, s_name, p_partkey limit 100;""")
            end2b=time.time()
            q2b=end2b-start2b
            run_times2b.append(q2b)
            print ("Elapsed time for the %d run of the 2nd query is : %s seconds" %(i,q2b))

        
    q2_coldb=run_times2b[0]
    q2_min_hotb=min(run_times2b[1:])
    q2_max_hotb=max(run_times2b[1:])
    q2_avgb=(sum(run_times2b[1:]))/(len(run_times2b)-1)
    print ("The cold run for the 2nd query was: %s seconds" %q2_coldb)
    print ("The fastest time for the 2nd query hot runs was : %s seconds" %q2_min_hotb)
    print ("The slowest time for 2nd query hot runs was : %s" %q2_max_hotb)
    print ("Average time for the 2nd query hot runs is : %s seconds" %q2_avgb)
    print ("---------------------------------------------------------------")
   

    run_times3b=[]
    for i in range(1,num_times):
            start3b=time.time()
            cur.execute("""Select l_orderkey,sum(l_extendedprice * (1-l_discount)) as revenue, o_orderdate,o_shippriority from customer,orders,lineitem where c_mktsegment='BUILDING' and c_custkey=o_custkey and l_orderkey=o_orderkey and o_orderdate<date '1995-03-15' and l_shipdate>date '1995-03-15' group by l_orderkey, o_orderdate, o_shippriority order by revenue desc, o_orderdate limit 10;""")
            end3b=time.time()
            q3b=end3b-start3b
            run_times3b.append(q3b)
            print ("Elapsed time for the %d run of the 3rd query is : %s seconds" %(i, q3b))

    q3_coldb=run_times3b[0]
    q3_min_hotb=min(run_times3b[1:])
    q3_max_hotb=max(run_times3b[1:])
    q3_avgb=(sum(run_times3b[1:]))/(len(run_times3b)-1)
    print ("The cold run for the 3rd query was: %s seconds" %q3_coldb)
    print ("The fastest time for the 3rd query hot runs was : %s seconds" %q3_min_hotb)
    print ("The slowest time for the 3rd query hot runs was : %s" %q3_max_hotb)
    print ("Average time for the 3rd query hot runs is : %s seconds" %q3_avgb)
    print ("---------------------------------------------------------------")
    
    run_times4b=[]
    for i in range(1,num_times):
            start4b=time.time()
            cur.execute("""Select o_orderpriority, count(*) as order_count from orders where o_orderdate>= date '1993-07-01' and o_orderdate < date '1993-07-01' + interval '3' month and exists ( Select * from lineitem where l_orderkey=o_orderkey and l_commitdate < l_receiptdate) group by o_orderpriority order by o_orderpriority;""")
            end4b=time.time()
            q4b=end4b-start4b
            run_times4b.append(q4b)
            print ("Elapsed time for the %d run of the 4th query is : %s seconds" % (i,q4b))

    q4_coldb=run_times4b[0]
    q4_min_hotb=min(run_times4b[1:])
    q4_max_hotb=max(run_times4b[1:])
    q4_avgb=(sum(run_times4b[1:]))/(len(run_times4b)-1)
    print ("The cold run for the 4th query was: %s seconds" %q4_cold)
    print ("The fastest time for the 4th query hot runs was : %s seconds" %q4_min_hot)
    print ("The slowest time for the 4th query runs was : %s" %q4_max_hot)
    print ("Average time for the 4th query hot runs is : %s seconds" %q4_avg)
    print ("---------------------------------------------------------------")

    run_times5b=[]
    for i in range(1, num_times):
            start5b = time.time()
            cur.execute("""Select n_name, sum(l_extendedprice * (1-l_discount)) as revenue from customer, orders, lineitem, supplier, nation, region where c_custkey=o_custkey and l_orderkey=o_orderkey and l_suppkey=s_suppkey and c_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='ASIA' and o_orderdate >=date '1994-01-01' and o_orderdate>=date '1994-01-01' and o_orderdate<date '1994-01-01' + interval '1' year group by n_name order by revenue desc;""")
            end5b=time.time()
            q5b = end5b - start5b
            run_times5b.append(q5b)
            print ("Elapsed time for the run %d of the 5th query is : %s seconds" % (i, q5b) )
        
    q5_coldb=run_times5b[0]
    q5_min_hotb=min(run_times5b[1:])
    q5_max_hotb=max(run_times5b[1:])
    q5_avgb=(sum(run_times5b[1:]))/(len(run_times5b)-1)
    print ("The cold run for the 5th query was: %s seconds" %q5_coldb)
    print ("The fastest time for the 5th query hot runs was : %s seconds" %q5_min_hotb)
    print ("The slowest time for the 5th hot runs was : %s" %q5_max_hotb)
    print ("Average time for the 5th query hot runs is : %s seconds" %q5_avgb)
    print ("---------------------------------------------------------------")
 
    run_times6b=[]
    for i in range(1,num_times):
            start6b=time.time()
            cur.execute("""Select sum(l_extendedprice * l_discount) as revenue from lineitem where l_shipdate>=date '1994-01-01' and l_shipdate < date '1994-01-01' + interval '1' year and l_discount between 0.06 - 0.01 and 0.06 + 0.01 and l_quantity<24;""")
            end6b=time.time()
            q6b=end6b-start6b
            run_times6b.append(q6b)
            print ("Elapsed time for the %d run of the 6th query is : %s seconds" % (i,q6b))

    q6_coldb=run_times6b[0]
    q6_min_hotb=min(run_times6b[1:])
    q6_max_hotb=max(run_times6b[1:])
    q6_avgb=(sum(run_times6b[1:]))/(len(run_times6b)-1)
    print ("The cold run for the 6th query was: %s seconds" %q6_coldb)
    print ("The fastest time for the 6th query hot runs was : %s seconds" %q6_min_hotb)
    print ("The slowest time for the 6th hot runs was : %s" %q6_max_hotb)
    print ("Average time for the 6th query hot runs is : %s seconds" %q6_avgb)
    print ("---------------------------------------------------------------")

    run_times7b=[]
    for i in range(1,num_times):
            start7b=time.time()
            cur.execute("""Select supp_nation,cust_nation,l_year,sum(volume) as revenue from ( Select n1.n_name as supp_nation, n2.n_name as cust_nation, extract(year from l_shipdate) as l_year, l_extendedprice * (1-l_discount) as volume from supplier,lineitem,orders,customer,nation n1, nation n2 where s_suppkey=l_suppkey and o_orderkey=l_orderkey and c_custkey=o_custkey and s_nationkey=n1.n_nationkey and c_nationkey=n2.n_nationkey and ((n1.n_name='FRANCE' and n2.n_name='GERMANY') or (n1.n_name='GERMANY' and n2.n_name='FRANCE')) and l_shipdate between date '1995-01-01' and date '1996-12-31') as shipping group by supp_nation,cust_nation,l_year order by supp_nation,cust_nation,l_year;""")
            end7b=time.time()
            q7b=end7b-start7b
            run_times7b.append(q7b)
            print ("Elapsed time for the %d run of the 7th query is : %s seconds" %(i, q7b))

    q7_coldb=run_times7b[0]
    q7_min_hotb=min(run_times7b[1:])
    q7_max_hotb=max(run_times7b[1:])
    q7_avgb=(sum(run_times7b[1:]))/(len(run_times7b)-1)
    print ("The cold run for the 7th query was: %s seconds" %q7_coldb)
    print ("The fastest time for the 7th query hot runs was : %s seconds" %q7_min_hotb)
    print ("The slowest time for the 7th hot runs was : %s" %q7_max_hotb)
    print ("Average time for the 7th query hot runs is : %s seconds" %q7_avgb)
    print ("---------------------------------------------------------------")
        
    run_times8b=[]
    for i in range(1,num_times):
            start8b=time.time()
            cur.execute("""Select o_year, sum(case when nation='BRAZIL' then volume else 0 end)/sum(volume) as mkt_share from (Select extract(year from o_orderdate) as o_year,l_extendedprice * (1-l_discount) as volume, n2.n_name as nation from part, supplier, lineitem, orders, customer, nation n1, nation n2, region where p_partkey=l_partkey and s_suppkey=l_suppkey and l_orderkey=o_orderkey and o_custkey=c_custkey and c_nationkey=n1.n_nationkey and n1.n_regionkey=r_regionkey and r_name='AMERICA' and s_nationkey=n2.n_nationkey and o_orderdate between date '1995-01-01' and date '1996-12-31' and p_type='ECONOMY ANODIZED STEEL') as all_nations group by o_year order by o_year;""")
            end8b=time.time()
            q8b=end8b-start8b
            run_times8b.append(q8b)
            print ("Elapsed time for the %d run of the 8th query is : %s seconds" %(i,q8b))

    q8_coldb=run_times8b[0]
    q8_min_hotb=min(run_times8b[1:])
    q8_max_hotb=max(run_times8b[1:])
    q8_avgb=(sum(run_times8b[1:]))/(len(run_times8b)-1)
    print ("The cold run for the 8th query was: %s seconds" %q8_coldb)
    print ("The fastest time for the 8th query hot runs was : %s seconds" %q8_min_hotb)
    print ("The slowest time for the 8th hot runs was : %s" %q8_max_hotb)
    print ("Average time for the 8th query hot runs is : %s seconds" %q8_avgb)
    print ("---------------------------------------------------------------")

    run_times9b=[]
    for i in range(1,num_times):
            start9b=time.time()
            cur.execute("""Select nation,o_year,sum(amount) as sum_profit from ( Select n_name as nation, extract(year from o_orderdate) as o_year, l_extendedprice * (1-l_discount)-ps_supplycost * l_quantity as amount from part, supplier, lineitem, partsupp, orders, nation where s_suppkey=l_suppkey and ps_suppkey=l_suppkey and ps_partkey=l_partkey and p_partkey=l_partkey and o_orderkey=l_orderkey and s_nationkey=n_nationkey and p_name like '%green%') as profit group by nation, o_year order by nation,o_year desc;""")
            end9b=time.time()
            q9b=end9b-start9b
            run_times9b.append(q9b)
            print ("Elapsed time for the %d run of the 9th query is : %s seconds" % (i,q9b))

    q9_coldb=run_times9b[0]
    q9_min_hotb=min(run_times9b[1:])
    q9_max_hotb=max(run_times9b[1:])
    q9_avgb=(sum(run_times9b[1:]))/(len(run_times9b)-1)
    print ("The cold run for the 9th query was: %s seconds" %q9_coldb)
    print ("The fastest time for the 9th query hot runs was : %s seconds" %q9_min_hotb)
    print ("The slowest time for the 9th hot runs was : %s" %q9_max_hotb)
    print ("Average time for the 9th query hot runs is : %s seconds" %q9_avgb)
    print ("---------------------------------------------------------------")
        
    run_times10b=[]
    for i in range(1,num_times):
            start10b=time.time()
            cur.execute("""Select c_custkey, c_name, sum(l_extendedprice * (1-l_discount)) as revenue, c_acctbal, n_name, c_address, c_phone, c_comment from customer, orders, lineitem, nation where c_custkey=o_custkey and l_orderkey=o_orderkey and o_orderdate>=date '1993-10-01' and o_orderdate < date '1993-10-01' + interval '3' month and l_returnflag='R' and c_nationkey=n_nationkey group by c_custkey, c_name, c_acctbal, c_phone, n_name, c_address, c_comment order by revenue desc limit 20;""")
            end10b=time.time()
            q10b=end10b-start10b
            run_times10b.append(q10b)
            print ("Elapsed time for the %d run of the 10th query is : %s seconds" %(i,q10b))

    q10_coldb=run_times10b[0]
    q10_min_hotb=min(run_times10b[1:])
    q10_max_hotb=max(run_times10b[1:])
    q10_avgb=(sum(run_times10b[1:]))/(len(run_times10b)-1)
    print ("The cold run for the 10th query was: %s seconds" %q10_coldb)
    print ("The fastest time for the 10th query hot runs was : %s seconds" %q10_min_hotb)
    print ("The slowest time for the 10th hot runs was : %s" %q10_max_hotb)
    print ("Average time for the 10th query hot runs is : %s seconds" %q10_avgb)
    print ("---------------------------------------------------------------")

    run_times11b=[]
    for i in range(1,num_times):
            start11b=time.time()
            cur.execute("""Select ps_partkey, sum(ps_supplycost * ps_availqty) as value from partsupp, supplier, nation where ps_suppkey=s_suppkey and s_nationkey=n_nationkey and n_name='GERMANY' group by ps_partkey having sum(ps_supplycost * ps_availqty)>(Select sum(ps_supplycost * ps_availqty) * 0.0001000000 from partsupp, supplier, nation where ps_suppkey=s_suppkey and s_nationkey=n_nationkey and n_name='GERMANY') order by value desc;""")
            end11b=time.time()
            q11b=end11b-start11b
            run_times11b.append(q11b)
            print ("Elapsed time for the %d run of the 11th query is : %s seconds" %(i, q11b))

    q11_coldb=run_times11b[0]
    q11_min_hotb=min(run_times11b[1:])
    q11_max_hotb=max(run_times11b[1:])
    q11_avgb=(sum(run_times11b[1:]))/(len(run_times11b)-1)
    print ("The cold run for the 11th query was: %s seconds" %q11_coldb)
    print ("The fastest time for the 11th query hot runs was : %s seconds" %q11_min_hotb)
    print ("The slowest time for the 11th hot runs was : %s" %q11_max_hotb)
    print ("Average time for the 11th query hot runs is : %s seconds" %q11_avgb)
    print ("---------------------------------------------------------------")

    run_times12b=[]
    for i in range(1,num_times):
            start12b=time.time()
            cur.execute("""Select l_shipmode,sum(case when o_orderpriority='1-URGENT' or o_orderpriority='2-HIGH' then 1 else 0 end) as high_line_count, sum(case when o_orderpriority <> '1=URGENT' and o_orderpriority <> '2-HIGH' then 1 else 0 end) as low_line_count from orders,lineitem where o_orderkey=l_orderkey and l_shipmode in('MAIL','SHIP') and l_commitdate < l_receiptdate and l_shipdate < l_commitdate and l_receiptdate >= date '1994-01-01' and l_receiptdate < date '1004-01-01'+ interval '1' year group by l_shipmode order by l_shipmode;""")
            end12b=time.time()
            q12b=end12b-start12b
            run_times12b.append(q12b)
            print ("Elapsed time for the %d run of the 12th query is : %s seconds" %(i,q12b))

    q12_coldb=run_times12b[0]
    q12_min_hotb=min(run_times12b[1:])
    q12_max_hotb=max(run_times12b[1:])
    q12_avgb=(sum(run_times12b[1:]))/(len(run_times12b)-1)
    print ("The cold run for the 12th query was: %s seconds" %q12_coldb)
    print ("The fastest time for the 12th query hot runs was : %s seconds" %q12_min_hotb)
    print ("The slowest time for the 12th hot runs was : %s" %q12_max_hotb)
    print ("Average time for the 12th query hot runs is : %s seconds" %q12_avgb)
    print ("---------------------------------------------------------------")

    run_times13b=[]
    for i in range(1,num_times):
            start13b=time.time()
            cur.execute("""Select c_count,count(*) as custdist from ( Select c_custkey, count(o_orderkey) from customer left outer join orders on c_custkey=o_custkey and o_comment not like '%special%requests%' group by c_custkey) as c_orders(c_custkey,c_count) group by c_count order by custdist desc, c_count desc;""")
            end13b=time.time()
            q13b=end13b-start13b
            run_times13b.append(q13b)
            print ("Elapsed time for the %d run of the 13th query is %s seconds" %(i,q13b))

    q13_coldb=run_times13b[0]
    q13_min_hotb=min(run_times13b[1:])
    q13_max_hotb=max(run_times13b[1:])
    q13_avgb=(sum(run_times13b[1:]))/(len(run_times13b)-1)
    print ("The cold run for the 13th query was: %s seconds" %q13_coldb)
    print ("The fastest time for the 13th query hot runs was : %s seconds" %q13_min_hotb)
    print ("The slowest time for the 13th hot runs was : %s" %q13_max_hotb)
    print ("Average time for the 13th query hot runs is : %s seconds" %q13_avgb)
    print ("---------------------------------------------------------------")

    run_times14b=[]
    for i in range(1,num_times):
            start14b=time.time()
            cur.execute("""Select 100.00 * sum(case when p_type like 'PROMO%' then l_extendedprice * (1-l_discount) else 0 end) / sum(l_extendedprice * (1-l_discount)) as promo_revenue from lineitem,part where l_partkey=p_partkey and l_shipdate >= date '1995-09-01' and l_shipdate < date '1995-09-01' + interval '1' month;""")
            end14b=time.time()
            q14b=end14b-start14b
            run_times14b.append(q14b)
            print ("Elapsed time for the %d run of the 14th query is %s seconds" %(i,q14b))

    q14_coldb=run_times14b[0]
    q14_min_hotb=min(run_times14b[1:])
    q14_max_hotb=max(run_times14b[1:])
    q14_avgb=(sum(run_times14b[1:]))/(len(run_times14b)-1)
    print ("The cold run for the 14th query was: %s seconds" %q14_coldb)
    print ("The fastest time for the 14th query hot runs was : %s seconds" %q14_min_hotb)
    print ("The slowest time for the 14th hot runs was : %s" %q14_max_hotb)
    print ("Average time for the 14th query hot runs is : %s seconds" %q14_avgb)
    print ("---------------------------------------------------------------")
    
    run_times15b=[]
    for i in range(1,num_times):
            start15b=time.time()
            cur.execute("""Create view revenue0(supplier_no, total_revenue) as select l_suppkey, sum(l_extendedprice * (1-l_discount)) from lineitem where l_shipdate >= date '1996-01-01' and l_shipdate < date '1996-01-01' + interval '3' month group by l_suppkey; Select s_suppkey,s_name, s_address, s_phone, total_revenue from supplier, revenue0 where s_suppkey=supplier_no and total_revenue=(select max(total_revenue) from revenue0) order by s_suppkey; drop view revenue0;""")
            end15b=time.time()
            q15b=end15b-start15b
            run_times15b.append(q15b)
            print ("Elapsed time for the %d run of the 15th query is %s seconds" %(i,q15b))

    q15_coldb=run_times15b[0]
    q15_min_hotb=min(run_times15b[1:])
    q15_max_hotb=max(run_times15b[1:])
    q15_avgb=(sum(run_times15b[1:]))/(len(run_times15b)-1)
    print ("The cold run for the 15th query was: %s seconds" %q15_coldb)
    print ("The fastest time for the 15th query hot runs was : %s seconds" %q15_min_hotb)
    print ("The slowest time for the 15th hot runs was : %s" %q15_max_hotb)
    print ("Average time for the 15th query hot runs is : %s seconds" %q15_avgb)
    print ("---------------------------------------------------------------")

    run_times16b=[]
    for i in range(1,num_times):
            start16b=time.time()
            cur.execute("""Select p_brand, p_type, p_size, count(distinct ps_suppkey) as supplier_cnt from partsupp, part where p_partkey=ps_partkey and p_brand <> 'Brand#45'and p_type not like 'MEDIUM POLISHED%' and p_size in (49,14,23,45,19,3,36,9) and ps_suppkey not in( Select s_suppkey from supplier where s_comment like '%Customer%Complaints%') group by p_brand,p_type,p_size order by supplier_cnt desc, p_brand,p_type, p_size;""")
            end16b=time.time()
            q16b=end16b-start16b
            run_times16b.append(q16b)
            print ("Elapsed time for the %d run of the 16th query is %s seconds" %(i,q16b))

    q16_coldb=run_times16b[0]
    q16_min_hotb=min(run_times16b[1:])
    q16_max_hotb=max(run_times16b[1:])
    q16_avgb=(sum(run_times16b[1:]))/(len(run_times16b)-1)
    print ("The cold run for the 16th query was: %s seconds" %q16_coldb)
    print ("The fastest time for the 16th query hot runs was : %s seconds" %q16_min_hotb)
    print ("The slowest time for the 16th hot runs was : %s" %q16_max_hotb)
    print ("Average time for the 16th query hot runs is : %s seconds" %q16_avgb)
    print ("---------------------------------------------------------------")

    run_times17b=[]
    for i in range(1,num_times):
            start17b=time.time()
            cur.execute("""Select sum(l_extendedprice) / 7.0 as avg_yearly from lineitem, part where p_partkey=l_partkey and p_brand='Brand#23' and p_container='MED BOX' and l_quantity < (Select 0.2 * avg(l_quantity) from lineitem where l_partkey=p_partkey);""")
            end17b=time.time()
            q17b=end17b-start17b
            run_times17b.append(q17b)
            print ("Elapsed time for the %d run of the 17th query is %s seconds" %(i,q17b))

    q17_coldb=run_times17b[0]
    q17_min_hotb=min(run_times17b[1:])
    q17_max_hotb=max(run_times17b[1:])
    q17_avgb=(sum(run_times17b[1:]))/(len(run_times17b)-1)
    print ("The cold run for the 17th query was: %s seconds" %q17_coldb)
    print ("The fastest time for the 17th query hot runs was : %s seconds" %q17_min_hotb)
    print ("The slowest time for the 17th hot runs was : %s" %q17_max_hotb)
    print ("Average time for the 17th query hot runs is : %s seconds" %q17_avgb)
    print ("---------------------------------------------------------------")

    run_times18b=[]
    for i in range(1,num_times):
            start18b=time.time()
            cur.execute("""Select c_name,c_custkey,o_orderkey, o_orderdate, o_totalprice, sum(l_quantity) from customer, orders, lineitem where o_orderkey in (Select l_orderkey from lineitem group by l_orderkey having sum(l_quantity)>300) and c_custkey=o_custkey and o_orderkey=l_orderkey group by c_name, c_custkey, o_orderkey,o_orderdate,o_totalprice order by o_totalprice desc, o_orderdate limit 100;""")
            end18b=time.time()
            q18b=end18b-start18b
            run_times18b.append(q18b)
            print ("Elapsed time for the %d run of the 18th query is %s seconds" %(i,q18b))

    q18_coldb=run_times18b[0]
    q18_min_hotb=min(run_times18b[1:])
    q18_max_hotb=max(run_times18b[1:])
    q18_avgb=(sum(run_times18b[1:]))/(len(run_times18b)-1)
    print ("The cold run for the 18th query was: %s seconds" %q18_coldb)
    print ("The fastest time for the 18th query hot runs was : %s seconds" %q18_min_hotb)
    print ("The slowest time for the 18th hot runs was : %s" %q18_max_hotb)
    print ("Average time for the 18th query hot runs is : %s seconds" %q18_avgb)
    print ("---------------------------------------------------------------")

    run_times19b=[]
    for i in range(1,num_times):
            start19b=time.time()
            cur.execute("""Select sum(l_extendedprice * (1-l_discount)) as revenrue from lineitem, part where ( p_partkey=l_partkey and p_brand='Brand#12' and p_container in ('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG') and l_quantity >=1 and l_quantity <= 1+10 and p_size between 1 and 5 and l_shipmode in ('AIR', 'AIR REG') and l_shipinstruct='DELIVER IN PERSON') or ( p_partkey=l_partkey and p_brand ='Brand#23' and p_container in ('MED BAG','MED BOX', 'MED PKG', 'MED PACK') and l_quantity >=10 and l_quantity <= 10+10 and p_size between 1 and 10 and l_shipmode in ('AIR','AIR REG') and l_shipinstruct='DELIVER IN PERSON') or (p_partkey=l_partkey and p_brand ='Brand#34' and p_container in ('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG') and l_quantity >= 20 and l_quantity <=20+10 and p_size between 1 and 15 and l_shipmode in ('AIR', 'AIR REG') and l_shipinstruct='DELIVER IN PERSON');""")
            end19b=time.time()
            q19b=end19b-start19b
            run_times19b.append(q19b)
            print ("Elapsed time for the %d run of the 19th query is %s seconds" %(i,q19b))

    q19_coldb=run_times19b[0]
    q19_min_hotb=min(run_times19b[1:])
    q19_max_hotb=max(run_times19b[1:])
    q19_avgb=(sum(run_times19b[1:]))/(len(run_times19b)-1)
    print ("The cold run for the 19th query was: %s seconds" %q19_coldb)
    print ("The fastest time for the 19th query hot runs was : %s seconds" %q19_min_hotb)
    print ("The slowest time for the 19th hot runs was : %s" %q19_max_hotb)
    print ("Average time for the 19th query hot runs is : %s seconds" %q19_avgb)
    print ("---------------------------------------------------------------")

    run_times20b=[]
    for i in range(1,num_times):
            start20b=time.time()
            cur.execute("""Select s_name, s_address from supplier, nation where s_suppkey in ( Select ps_suppkey from partsupp where ps_partkey in (Select p_partkey from part where p_name like 'forest%' ) and ps_availqty > ( Select 0.5 * sum(l_quantity) from lineitem where l_partkey=ps_partkey and l_suppkey = ps_suppkey and l_shipdate >= date '1994-01-01' and l_shipdate < date '1994-01-01' + interval '1' year)) and s_nationkey=n_nationkey and s_name ='CANADA' order by s_name;""")
            end20b=time.time()
            q20b=end20b-start20b
            run_times20b.append(q20b)
            print ("Elapsed time for the %d run of the 20th query is %s seconds" %(i,q20b))

    q20_coldb=run_times20b[0]
    q20_min_hotb=min(run_times20b[1:])
    q20_max_hotb=max(run_times20b[1:])
    q20_avgb=(sum(run_times20b[1:]))/(len(run_times20b)-1)
    print ("The cold run for the 20th query was: %s seconds" %q20_coldb)
    print ("The fastest time for the 20th query hot runs was : %s seconds" %q20_min_hotb)
    print ("The slowest time for the 20th hot runs was : %s" %q20_max_hotb)
    print ("Average time for the 20th query hot runs is : %s seconds" %q20_avgb)
    print ("---------------------------------------------------------------")

    run_times21b=[]
    for i in range(1, num_times):
            start21b=time.time()
            cur.execute("""Select s_name, count(*) as numwait from supplier, lineitem l1, orders, nation where s_suppkey=l1.l_suppkey and o_orderkey=l1.l_orderkey and o_orderstatus='F' and l1.l_receiptdate > l1.l_commitdate and exists ( Select * from lineitem l2 where l2.l_orderkey=l1.l_orderkey and l2.l_suppkey <> l1.l_suppkey) and not exists ( Select * from lineitem l3 where l3.l_orderkey =l1.l_orderkey and l3.l_suppkey<> l1.l_suppkey and l3.l_receiptdate > l3.l_commitdate) and s_nationkey=n_nationkey and n_name='SAUDI ARABIA' group by s_name order by numwait desc, s_name limit 100;""")
            end21b=time.time()
            q21b=end21b-start21b
            run_times21b.append(q21b)
            print ("Elapsed time for the %d run of the 21st query is %s seconds" %(i,q21b))

    q21_coldb=run_times21b[0]
    q21_min_hotb=min(run_times21b[1:])
    q21_max_hotb=max(run_times21b[1:])
    q21_avgb=(sum(run_times21b[1:]))/(len(run_times21b)-1)
    print ("The cold run for the 21st query was: %s seconds" %q21_coldb)
    print ("The fastest time for the 21st query hot runs was : %s seconds" %q21_min_hotb)
    print ("The slowest time for the 21st hot runs was : %s" %q21_max_hotb)
    print ("Average time for the 21st query hot runs is : %s seconds" %q21_avgb)
    print ("---------------------------------------------------------------")

    run_times22b=[]
    for i in range(1, num_times):
            start22b=time.time()
            cur.execute("""Select cntrycode, count(*) as numcust, sum(c_acctbal) as totacctbal from ( Select substring(c_phone from 1 for 2) as cntrycode,c_acctbal from customer where substring(c_phone from 1 for 2) in ('13','31','23','29','30','18','17') and c_acctbal > ( Select avg(c_acctbal) from customer where c_acctbal >0.00 and substring (c_phone from 1 for 2) in ('13','31','23','29','30','18','17')) and not exists ( Select * from orders where o_custkey=c_custkey)) as custsale group by cntrycode order by cntrycode;""")
            end22b=time.time()
            q22b=end22b-start22b
            run_times22b.append(q22b)
            print ("Elapsed time for the %d run of the 22nd query is %s seconds" %(i,q22b))

    q22_coldb=run_times22b[0]
    q22_min_hotb=min(run_times22b[1:])
    q22_max_hotb=max(run_times22b[1:])
    q22_avgb=(sum(run_times22b[1:]))/(len(run_times22b)-1)
    print ("The cold run for the 22nd query was: %s seconds" %q22_coldb)
    print ("The fastest time for the 22nd query hot runs was : %s seconds" %q22_min_hotb)
    print ("The slowest time for the 22nd hot runs was : %s" %q22_max_hotb)
    print ("Average time for the 22nd query hot runs is : %s seconds" %q22_avgb)
    print ("---------------------------------------------------------------")

    ListBeforeUpd=[q1_cold,q1_min_hot,q1_max_hot,q1_avg,q2_cold,q2_min_hot,q2_max_hot,q2_avg,q3_cold,q3_min_hot,q3_max_hot,q3_avg,q4_cold,q4_min_hot,q4_max_hot,q4_avg,q5_cold,q5_min_hot,q5_max_hot,q5_avg,q6_cold,q6_min_hot,q6_max_hot,q6_avg,q7_cold,q7_min_hot,q7_max_hot,q7_avg,q8_cold,q8_min_hot,q8_max_hot,q8_avg,q9_cold,q9_min_hot,q9_max_hot,q9_avg,q10_cold,q10_min_hot,q10_max_hot,q10_avg,q11_cold,q11_min_hot,q11_max_hot,q11_avg,q12_cold,q12_min_hot,q12_max_hot,q12_avg,q13_cold,q13_min_hot,q13_max_hot,q13_avg,q14_cold,q14_min_hot,q14_max_hot,q14_avg,q15_cold,q15_min_hot,q15_max_hot,q15_avg,q16_cold,q16_min_hot,q16_max_hot,q16_avg,q17_cold,q17_min_hot,q17_max_hot,q17_avg,q18_cold,q18_min_hot,q18_max_hot,q18_avg,q19_cold,q19_min_hot,q19_max_hot,q19_avg,q20_cold,q20_min_hot,q20_max_hot,q20_avg,q21_cold,q21_min_hot,q21_max_hot,q21_avg,q22_cold,q22_min_hot,q22_max_hot,q22_avg]

    ListAfterUpd=[q1_coldb,q1_min_hotb,q1_max_hotb,q1_avgb,q2_coldb,q2_min_hotb,q2_max_hotb,q2_avgb,q3_coldb,q3_min_hotb,q3_max_hotb,q3_avgb,q4_coldb,q4_min_hotb,q4_max_hotb,q4_avgb,q5_coldb,q5_min_hotb,q5_max_hotb,q5_avgb,q6_coldb,q6_min_hotb,q6_max_hotb,q6_avgb,q7_coldb,q7_min_hotb,q7_max_hotb,q7_avgb,q8_coldb,q8_min_hotb,q8_max_hotb,q8_avgb,q9_coldb,q9_min_hotb,q9_max_hotb,q9_avgb,q10_coldb,q10_min_hotb,q10_max_hotb,q10_avgb,q11_coldb,q11_min_hotb,q11_max_hotb,q11_avgb,q12_coldb,q12_min_hotb,q12_max_hotb,q12_avgb,q13_coldb,q13_min_hotb,q13_max_hotb,q13_avgb,q14_coldb,q14_min_hotb,q14_max_hotb,q14_avgb,q15_coldb,q15_min_hotb,q15_max_hotb,q15_avgb,q16_coldb,q16_min_hotb,q16_max_hotb,q16_avgb,q17_coldb,q17_min_hotb,q17_max_hotb,q17_avgb,q18_coldb,q18_min_hotb,q18_max_hotb,q18_avgb,q19_coldb,q19_min_hotb,q19_max_hotb,q19_avgb,q20_coldb,q20_min_hotb,q20_max_hotb,q20_avgb,q21_coldb,q21_min_hotb,q21_max_hotb,q21_avgb,q22_coldb,q22_min_hotb,q22_max_hotb,q22_avgb]
    

    listfin=map(sub,ListAfterUpd,ListBeforeUpd)
    for i in xrange(0, len(listfin), 4):
        print "Results of the %d query for the cold run difference : %s \n minimum hot run difference : %s \n Maximum hot difference : %s \n Average hot runs difference : %s" %((i/4)+1, listfin[i], listfin[i + 1], listfin[i + 2], listfin[i + 3])

else:
    print "Installing PostgreSQL.."
    os.chdir(home)
    os.mkdir(Dirvar)
    os.chdir(Dirvar)
    
    
    os.system("git clone https://github.com/postgres/postgres")
    os.chdir("postgres")
    PREFIX=os.path.join(home,Dirvar)
    os.system("./configure --prefix=%s" % PREFIX)
    os.system("make")
    os.system("make install")
    os.chdir(home)
    os.chdir(Dirvar)
    os.chdir("bin")

    Datadir=raw_input("Please identify the directory where the data for this database will reside: ")
    while os.path.exists(os.path.join(home,Dirvar,Datadir))==True:
        Datadir=raw_input("The directory already exists, please specify the directory where the data for this database will reside: ")
    else:
	os.system("./initdb -D %s" % Datadir)
	print "Starting PostgreSQL.."

    os.system("./pg_ctl -D %s -l logfile start" % Datadir)
    DBname=raw_input("Please specify the name of your database: ")
    os.system("./createdb %s" % DBname)

    conn=psycopg2.connect("dbname='%s' user='%s' host='localhost'" % (DBname,Uservar))
 
    cur=conn.cursor()
    cur.execute("""Create table nation (n_nationkey integer, n_name text, n_regionkey integer, n_comment text)""")
    cur.execute("""Create table supplier (s_suppkey integer, s_name text, s_address text, s_nationkey integer, s_phone text, s_acctbal decimal, s_comment text)""")
    cur.execute("""Create table region (r_regionkey integer, r_name text, r_comment text)""")

    cur.execute( """Create table customer (c_custkey integer,c_name text, c_address text, c_nationkey integer, c_phone text, c_acctbal decimal, c_mktsegment text, c_comment text)""")
    cur.execute("""Create table orders (o_orderkey integer, o_custkey integer, o_orderstatus text,o_totalprice decimal, o_orderdate date, o_orderpriority text, o_shippriority text, o_comment text, extracol text)""")

    cur.execute(""" Create table part(p_partkey integer, p_name text, p_mfgr text, p_brand text, p_type text, p_size integer, p_container text, p_retailprice decimal, p_comment text)""")
    cur.execute("""Create table partsupp (ps_partkey integer, ps_suppkey integer, ps_availqty integer, ps_supplycost decimal, ps_comment text)""")

    cur.execute("""Create table lineitem (l_orderkey integer, l_partkey integer, l_suppkey integer, l_linenumber integer, l_quantity real, l_extendedprice decimal, l_discount decimal, l_tax decimal,l_returnflag text, l_linestatus text, l_shipdate date, l_commitdate date, l_receiptdate date, l_shipinstruct text, l_shipmode text, l_comment text)""")
    print "Loading the tables"
    startload2=time.time()
    cur.execute("""Copy customer from '%s' (format csv, delimiter ('|'));""" % pathcustomer)

    cur.execute("""Copy lineitem from '%s' (format csv, delimiter ('|'));""" % pathlineitem)

    cur.execute("""Copy nation from '%s' (format csv, delimiter ('|'));""" % pathnation)

    cur.execute("""Copy orders from '%s' (format csv, delimiter ('|'));""" % pathorders)

    cur.execute("""Copy part  from '%s' (format csv, delimiter ('|'));""" % pathpart)

    cur.execute("""Copy partsupp from '%s' (format csv, delimiter ('|'));""" % pathpartsupp)

    cur.execute("""Copy region from '%s' (format csv, delimiter ('|'));""" % pathregion)

    cur.execute("""Copy supplier from '%s' (format csv, delimiter ('|'));""" % pathsupplier)
    endload2=time.time()
    difload2=endload2-startload2
    print "Done loading the db"
    print ("Time loading the tables : %s seconds" %difload2)
    num_times=6
    run_times1=[]
    for i in range(1,num_times):
            start1=time.time()
            cur.execute("""Select l_returnflag,l_linestatus,sum(l_quantity) as sum_qty, sum(l_extendedprice) as sum_base_price,sum(l_extendedprice * (1 - l_discount)) as sum_disc_price,sum(l_extendedprice * (1-l_discount) * (1 + l_tax)) as sum_charge, avg(l_quantity) as avg_qty,avg(l_extendedprice) as avg_price, avg(l_discount) as avg_disc,count(*) as count_order from lineitem where l_shipdate <= date '1998-12-01' - interval '90' day group by l_returnflag,l_linestatus order by l_returnflag,l_linestatus;""")
            end1=time.time()
            q1=end1-start1
            run_times1.append(q1)
            print ("Elapsed time for the %d run of the 1st query is : %s seconds" %(i,q1))
    
    q1_cold=run_times1[0]
    
    q1_min_hot=min(run_times1[1:])
    q1_max_hot=max(run_times1[1:])
    q1_avg=(sum(run_times1[1:]))/(len(run_times1)-1)
    print ("The cold run for the 1st query was: %s seconds" %q1_cold)
    print ("The fastest time for the 1st query hot runs was : %s seconds" %q1_min_hot)
    print ("The slowest time for the hot runs was : %s" %q1_max_hot)
    print ("Average time for the 1st query hot runs is : %s seconds" %q1_avg)
    print ("---------------------------------------------------------------")


    run_times2=[]
    for i in range(1,num_times):
            start2=time.time()
            cur.execute("""Select s_acctbal,s_name,n_name,p_partkey,p_mfgr,s_address,s_phone,s_comment from part,supplier,partsupp,nation,region where p_partkey=ps_partkey and s_suppkey=ps_suppkey and p_size=15 and p_type like '%BRASS' and s_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='EUROPE' and ps_supplycost=(Select min(ps_supplycost) from partsupp,supplier,nation,region where p_partkey=ps_partkey and s_suppkey=ps_suppkey and s_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='EUROPE') order by s_acctbal desc, n_name, s_name, p_partkey limit 100;""")
            end2=time.time()
            q2=end2-start2
            run_times2.append(q2)
            print ("Elapsed time for the %d run of the 2nd query is : %s seconds" %(i,q2))

        
    q2_cold=run_times2[0]
    q2_min_hot=min(run_times2[1:])
    q2_max_hot=max(run_times2[1:])
    q2_avg=(sum(run_times2[1:]))/(len(run_times2)-1)
    print ("The cold run for the 2nd query was: %s seconds" %q2_cold)
    print ("The fastest time for the 2nd query hot runs was : %s seconds" %q2_min_hot)
    print ("The slowest time for 2nd query hot runs was : %s" %q2_max_hot)
    print ("Average time for the 2nd query hot runs is : %s seconds" %q2_avg)
    print ("---------------------------------------------------------------")
    run_times3=[]
    for i in range(1,num_times):
            start3=time.time()
            cur.execute("""Select l_orderkey,sum(l_extendedprice * (1-l_discount)) as revenue, o_orderdate,o_shippriority from customer,orders,lineitem where c_mktsegment='BUILDING' and c_custkey=o_custkey and l_orderkey=o_orderkey and o_orderdate<date '1995-03-15' and l_shipdate>date '1995-03-15' group by l_orderkey, o_orderdate, o_shippriority order by revenue desc, o_orderdate limit 10;""")
            end3=time.time()
            q3=end3-start3
            run_times3.append(q3)
            print ("Elapsed time for the %d run of the 3rd query is : %s seconds" %(i, q3))

    q3_cold=run_times3[0]
    q3_min_hot=min(run_times3[1:])
    q3_max_hot=max(run_times3[1:])
    q3_avg=(sum(run_times3[1:]))/(len(run_times3)-1)
    print ("The cold run for the 3rd query was: %s seconds" %q3_cold)
    print ("The fastest time for the 3rd query hot runs was : %s seconds" %q3_min_hot)
    print ("The slowest time for the 3rd query hot runs was : %s" %q3_max_hot)
    print ("Average time for the 3rd query hot runs is : %s seconds" %q3_avg)
    print ("---------------------------------------------------------------")

    run_times4=[]
    for i in range(1,num_times):
            start4=time.time()
            cur.execute("""Select o_orderpriority, count(*) as order_count from orders where o_orderdate>= date '1993-07-01' and o_orderdate < date '1993-07-01' + interval '3' month and exists ( Select * from lineitem where l_orderkey=o_orderkey and l_commitdate < l_receiptdate) group by o_orderpriority order by o_orderpriority;""")
            end4=time.time()
            q4=end4-start4
            run_times4.append(q4)
            print ("Elapsed time for the %d run of the 4th query is : %s seconds" % (i,q4))

    q4_cold=run_times4[0]
    q4_min_hot=min(run_times4[1:])
    q4_max_hot=max(run_times4[1:])
    q4_avg=(sum(run_times4[1:]))/(len(run_times4)-1)
    print ("The cold run for the 4th query was: %s seconds" %q4_cold)
    print ("The fastest time for the 4th query hot runs was : %s seconds" %q4_min_hot)
    print ("The slowest time for the 4th query runs was : %s" %q4_max_hot)
    print ("Average time for the 4th query hot runs is : %s seconds" %q4_avg)
    print ("---------------------------------------------------------------")
        
    run_times5=[]
    for i in range(1, num_times):
            start5 = time.time()
            cur.execute("""Select n_name, sum(l_extendedprice * (1-l_discount)) as revenue from customer, orders, lineitem, supplier, nation, region where c_custkey=o_custkey and l_orderkey=o_orderkey and l_suppkey=s_suppkey and c_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='ASIA' and o_orderdate >=date '1994-01-01' and o_orderdate>=date '1994-01-01' and o_orderdate<date '1994-01-01' + interval '1' year group by n_name order by revenue desc;""")
            end5=time.time()
            q5 = end5 - start5
            run_times5.append(q5)
            print ("Elapsed time for the run %d of the 5th query is : %s seconds" % (i, q5) )
        
    q5_cold=run_times5[0]
    q5_min_hot=min(run_times5[1:])
    q5_max_hot=max(run_times5[1:])
    q5_avg=(sum(run_times5[1:]))/(len(run_times5)-1)
    print ("The cold run for the 5th query was: %s seconds" %q5_cold)
    print ("The fastest time for the 5th query hot runs was : %s seconds" %q5_min_hot)
    print ("The slowest time for the 5th hot runs was : %s" %q5_max_hot)
    print ("Average time for the 5th query hot runs is : %s seconds" %q5_avg)
    print ("---------------------------------------------------------------")
 
    run_times6=[]
    for i in range(1,num_times):
            start6=time.time()
            cur.execute("""Select sum(l_extendedprice * l_discount) as revenue from lineitem where l_shipdate>=date '1994-01-01' and l_shipdate < date '1994-01-01' + interval '1' year and l_discount between 0.06 - 0.01 and 0.06 + 0.01 and l_quantity<24;""")
            end6=time.time()
            q6=end6-start6
            run_times6.append(q6)
            print ("Elapsed time for the %d run of the 6th query is : %s seconds" % (i,q6))

    q6_cold=run_times6[0]
    q6_min_hot=min(run_times6[1:])
    q6_max_hot=max(run_times6[1:])
    q6_avg=(sum(run_times6[1:]))/(len(run_times6)-1)
    print ("The cold run for the 6th query was: %s seconds" %q6_cold)
    print ("The fastest time for the 6th query hot runs was : %s seconds" %q6_min_hot)
    print ("The slowest time for the 6th hot runs was : %s" %q6_max_hot)
    print ("Average time for the 6th query hot runs is : %s seconds" %q6_avg)
    print ("---------------------------------------------------------------")

    run_times7=[]
    for i in range(1,num_times):
            start7=time.time()
            cur.execute("""Select supp_nation,cust_nation,l_year,sum(volume) as revenue from ( Select n1.n_name as supp_nation, n2.n_name as cust_nation, extract(year from l_shipdate) as l_year, l_extendedprice * (1-l_discount) as volume from supplier,lineitem,orders,customer,nation n1, nation n2 where s_suppkey=l_suppkey and o_orderkey=l_orderkey and c_custkey=o_custkey and s_nationkey=n1.n_nationkey and c_nationkey=n2.n_nationkey and ((n1.n_name='FRANCE' and n2.n_name='GERMANY') or (n1.n_name='GERMANY' and n2.n_name='FRANCE')) and l_shipdate between date '1995-01-01' and date '1996-12-31') as shipping group by supp_nation,cust_nation,l_year order by supp_nation,cust_nation,l_year;""")
            end7=time.time()
            q7=end7-start7
            run_times7.append(q7)
            print ("Elapsed time for the %d run of the 7th query is : %s seconds" %(i, q7))

    q7_cold=run_times7[0]
    q7_min_hot=min(run_times7[1:])
    q7_max_hot=max(run_times7[1:])
    q7_avg=(sum(run_times7[1:]))/(len(run_times7)-1)
    print ("The cold run for the 7th query was: %s seconds" %q7_cold)
    print ("The fastest time for the 7th query hot runs was : %s seconds" %q7_min_hot)
    print ("The slowest time for the 7th hot runs was : %s" %q7_max_hot)
    print ("Average time for the 7th query hot runs is : %s seconds" %q7_avg)
    print ("---------------------------------------------------------------")

    run_times8=[]
    for i in range(1,num_times):
            start8=time.time()
            cur.execute("""Select o_year, sum(case when nation='BRAZIL' then volume else 0 end)/sum(volume) as mkt_share from (Select extract(year from o_orderdate) as o_year,l_extendedprice * (1-l_discount) as volume, n2.n_name as nation from part, supplier, lineitem, orders, customer, nation n1, nation n2, region where p_partkey=l_partkey and s_suppkey=l_suppkey and l_orderkey=o_orderkey and o_custkey=c_custkey and c_nationkey=n1.n_nationkey and n1.n_regionkey=r_regionkey and r_name='AMERICA' and s_nationkey=n2.n_nationkey and o_orderdate between date '1995-01-01' and date '1996-12-31' and p_type='ECONOMY ANODIZED STEEL') as all_nations group by o_year order by o_year;""")
            end8=time.time()
            q8=end8-start8
            run_times8.append(q8)
            print ("Elapsed time for the %d run of the 8th query is : %s seconds" %(i,q8))

    q8_cold=run_times8[0]
    q8_min_hot=min(run_times8[1:])
    q8_max_hot=max(run_times8[1:])
    q8_avg=(sum(run_times8[1:]))/(len(run_times8)-1)
    print ("The cold run for the 8th query was: %s seconds" %q8_cold)
    print ("The fastest time for the 8th query hot runs was : %s seconds" %q8_min_hot)
    print ("The slowest time for the 8th hot runs was : %s" %q8_max_hot)
    print ("Average time for the 8th query hot runs is : %s seconds" %q8_avg)
    print ("---------------------------------------------------------------")

    run_times9=[]
    for i in range(1,num_times):
            start9=time.time()
            cur.execute("""Select nation,o_year,sum(amount) as sum_profit from ( Select n_name as nation, extract(year from o_orderdate) as o_year, l_extendedprice * (1-l_discount)-ps_supplycost * l_quantity as amount from part, supplier, lineitem, partsupp, orders, nation where s_suppkey=l_suppkey and ps_suppkey=l_suppkey and ps_partkey=l_partkey and p_partkey=l_partkey and o_orderkey=l_orderkey and s_nationkey=n_nationkey and p_name like '%green%') as profit group by nation, o_year order by nation,o_year desc;""")
            end9=time.time()
            q9=end9-start9
            run_times9.append(q9)
            print ("Elapsed time for the %d run of the 9th query is : %s seconds" % (i,q9))

    q9_cold=run_times9[0]
    q9_min_hot=min(run_times9[1:])
    q9_max_hot=max(run_times9[1:])
    q9_avg=(sum(run_times9[1:]))/(len(run_times9)-1)
    print ("The cold run for the 9th query was: %s seconds" %q9_cold)
    print ("The fastest time for the 9th query hot runs was : %s seconds" %q9_min_hot)
    print ("The slowest time for the 9th hot runs was : %s" %q9_max_hot)
    print ("Average time for the 9th query hot runs is : %s seconds" %q9_avg)
    print ("---------------------------------------------------------------")
        
    run_times10=[]
    for i in range(1,num_times):
            start10=time.time()
            cur.execute("""Select c_custkey, c_name, sum(l_extendedprice * (1-l_discount)) as revenue, c_acctbal, n_name, c_address, c_phone, c_comment from customer, orders, lineitem, nation where c_custkey=o_custkey and l_orderkey=o_orderkey and o_orderdate>=date '1993-10-01' and o_orderdate < date '1993-10-01' + interval '3' month and l_returnflag='R' and c_nationkey=n_nationkey group by c_custkey, c_name, c_acctbal, c_phone, n_name, c_address, c_comment order by revenue desc limit 20;""")
            end10=time.time()
            q10=end10-start10
            run_times10.append(q10)
            print ("Elapsed time for the %d run of the 10th query is : %s seconds" %(i,q10))

    q10_cold=run_times10[0]
    q10_min_hot=min(run_times10[1:])
    q10_max_hot=max(run_times10[1:])
    q10_avg=(sum(run_times10[1:]))/(len(run_times10)-1)
    print ("The cold run for the 10th query was: %s seconds" %q10_cold)
    print ("The fastest time for the 10th query hot runs was : %s seconds" %q10_min_hot)
    print ("The slowest time for the 10th hot runs was : %s" %q10_max_hot)
    print ("Average time for the 10th query hot runs is : %s seconds" %q10_avg)
    print ("---------------------------------------------------------------")

    run_times11=[]
    for i in range(1,num_times):
            start11=time.time()
            cur.execute("""Select ps_partkey, sum(ps_supplycost * ps_availqty) as value from partsupp, supplier, nation where ps_suppkey=s_suppkey and s_nationkey=n_nationkey and n_name='GERMANY' group by ps_partkey having sum(ps_supplycost * ps_availqty)>(Select sum(ps_supplycost * ps_availqty) * 0.0001000000 from partsupp, supplier, nation where ps_suppkey=s_suppkey and s_nationkey=n_nationkey and n_name='GERMANY') order by value desc;""")
            end11=time.time()
            q11=end11-start11
            run_times11.append(q11)
            print ("Elapsed time for the %d run of the 11th query is : %s seconds" %(i, q11))

    q11_cold=run_times11[0]
    q11_min_hot=min(run_times11[1:])
    q11_max_hot=max(run_times11[1:])
    q11_avg=(sum(run_times11[1:]))/(len(run_times11)-1)
    print ("The cold run for the 11th query was: %s seconds" %q11_cold)
    print ("The fastest time for the 11th query hot runs was : %s seconds" %q11_min_hot)
    print ("The slowest time for the 11th hot runs was : %s" %q11_max_hot)
    print ("Average time for the 11th query hot runs is : %s seconds" %q11_avg)
    print ("---------------------------------------------------------------")
       
    run_times12=[]
    for i in range(1,num_times):
            start12=time.time()
            cur.execute("""Select l_shipmode,sum(case when o_orderpriority='1-URGENT' or o_orderpriority='2-HIGH' then 1 else 0 end) as high_line_count, sum(case when o_orderpriority <> '1=URGENT' and o_orderpriority <> '2-HIGH' then 1 else 0 end) as low_line_count from orders,lineitem where o_orderkey=l_orderkey and l_shipmode in('MAIL','SHIP') and l_commitdate < l_receiptdate and l_shipdate < l_commitdate and l_receiptdate >= date '1994-01-01' and l_receiptdate < date '1004-01-01'+ interval '1' year group by l_shipmode order by l_shipmode;""")
            end12=time.time()
            q12=end12-start12
            run_times12.append(q12)
            print ("Elapsed time for the %d run of the 12th query is : %s seconds" %(i,q12))

    q12_cold=run_times12[0]
    q12_min_hot=min(run_times12[1:])
    q12_max_hot=max(run_times12[1:])
    q12_avg=(sum(run_times12[1:]))/(len(run_times12)-1)
    print ("The cold run for the 12th query was: %s seconds" %q12_cold)
    print ("The fastest time for the 12th query hot runs was : %s seconds" %q12_min_hot)
    print ("The slowest time for the 12th hot runs was : %s" %q12_max_hot)
    print ("Average time for the 12th query hot runs is : %s seconds" %q12_avg)
    print ("---------------------------------------------------------------")

    run_times13=[]
    for i in range(1,num_times):
            start13=time.time()
            cur.execute("""Select c_count,count(*) as custdist from ( Select c_custkey, count(o_orderkey) from customer left outer join orders on c_custkey=o_custkey and o_comment not like '%special%requests%' group by c_custkey) as c_orders(c_custkey,c_count) group by c_count order by custdist desc, c_count desc;""")
            end13=time.time()
            q13=end13-start13
            run_times13.append(q13)
            print ("Elapsed time for the %d run of the 13th query is %s seconds" %(i,q13))

    q13_cold=run_times13[0]
    q13_min_hot=min(run_times13[1:])
    q13_max_hot=max(run_times13[1:])
    q13_avg=(sum(run_times13[1:]))/(len(run_times13)-1)
    print ("The cold run for the 13th query was: %s seconds" %q13_cold)
    print ("The fastest time for the 13th query hot runs was : %s seconds" %q13_min_hot)
    print ("The slowest time for the 13th hot runs was : %s" %q13_max_hot)
    print ("Average time for the 13th query hot runs is : %s seconds" %q13_avg)
    print ("---------------------------------------------------------------")
    
    run_times14=[]
    for i in range(1,num_times):
            start14=time.time()
            cur.execute("""Select 100.00 * sum(case when p_type like 'PROMO%' then l_extendedprice * (1-l_discount) else 0 end) / sum(l_extendedprice * (1-l_discount)) as promo_revenue from lineitem,part where l_partkey=p_partkey and l_shipdate >= date '1995-09-01' and l_shipdate < date '1995-09-01' + interval '1' month;""")
            end14=time.time()
            q14=end14-start14
            run_times14.append(q14)
            print ("Elapsed time for the %d run of the 14th query is %s seconds" %(i,q14))

    q14_cold=run_times14[0]
    q14_min_hot=min(run_times14[1:])
    q14_max_hot=max(run_times14[1:])
    q14_avg=(sum(run_times14[1:]))/(len(run_times14)-1)
    print ("The cold run for the 14th query was: %s seconds" %q14_cold)
    print ("The fastest time for the 14th query hot runs was : %s seconds" %q14_min_hot)
    print ("The slowest time for the 14th hot runs was : %s" %q14_max_hot)
    print ("Average time for the 14th query hot runs is : %s seconds" %q14_avg)
    print ("---------------------------------------------------------------")
    
    run_times15=[]
    for i in range(1,num_times):
            start15=time.time()
            cur.execute("""Create view revenue0(supplier_no, total_revenue) as select l_suppkey, sum(l_extendedprice * (1-l_discount)) from lineitem where l_shipdate >= date '1996-01-01' and l_shipdate < date '1996-01-01' + interval '3' month group by l_suppkey; Select s_suppkey,s_name, s_address, s_phone, total_revenue from supplier, revenue0 where s_suppkey=supplier_no and total_revenue=(select max(total_revenue) from revenue0) order by s_suppkey; drop view revenue0;""")
            end15=time.time()
            q15=end15-start15
            run_times15.append(q15)
            print ("Elapsed time for the %d run of the 15th query is %s seconds" %(i,q15))

    q15_cold=run_times15[0]
    q15_min_hot=min(run_times15[1:])
    q15_max_hot=max(run_times15[1:])
    q15_avg=(sum(run_times15[1:]))/(len(run_times15)-1)
    print ("The cold run for the 15th query was: %s seconds" %q15_cold)
    print ("The fastest time for the 15th query hot runs was : %s seconds" %q15_min_hot)
    print ("The slowest time for the 15th hot runs was : %s" %q15_max_hot)
    print ("Average time for the 15th query hot runs is : %s seconds" %q15_avg)
    print ("---------------------------------------------------------------")

    run_times16=[]
    for i in range(1,num_times):
            start16=time.time()
            cur.execute("""Select p_brand, p_type, p_size, count(distinct ps_suppkey) as supplier_cnt from partsupp, part where p_partkey=ps_partkey and p_brand <> 'Brand#45'and p_type not like 'MEDIUM POLISHED%' and p_size in (49,14,23,45,19,3,36,9) and ps_suppkey not in( Select s_suppkey from supplier where s_comment like '%Customer%Complaints%') group by p_brand,p_type,p_size order by supplier_cnt desc, p_brand,p_type, p_size;""")
            end16=time.time()
            q16=end16-start16
            run_times16.append(q16)
            print ("Elapsed time for the %d run of the 16th query is %s seconds" %(i,q16))

    q16_cold=run_times16[0]
    q16_min_hot=min(run_times16[1:])
    q16_max_hot=max(run_times16[1:])
    q16_avg=(sum(run_times16[1:]))/(len(run_times16)-1)
    print ("The cold run for the 16th query was: %s seconds" %q16_cold)
    print ("The fastest time for the 16th query hot runs was : %s seconds" %q16_min_hot)
    print ("The slowest time for the 16th hot runs was : %s" %q16_max_hot)
    print ("Average time for the 16th query hot runs is : %s seconds" %q16_avg)
    print ("---------------------------------------------------------------")

    run_times17=[]
    for i in range(1,num_times):
            start17=time.time()
            cur.execute("""Select sum(l_extendedprice) / 7.0 as avg_yearly from lineitem, part where p_partkey=l_partkey and p_brand='Brand#23' and p_container='MED BOX' and l_quantity < (Select 0.2 * avg(l_quantity) from lineitem where l_partkey=p_partkey);""")
            end17=time.time()
            q17=end17-start17
            run_times17.append(q17)
            print ("Elapsed time for the %d run of the 17th query is %s seconds" %(i,q17))

    q17_cold=run_times17[0]
    q17_min_hot=min(run_times17[1:])
    q17_max_hot=max(run_times17[1:])
    q17_avg=(sum(run_times17[1:]))/(len(run_times17)-1)
    print ("The cold run for the 17th query was: %s seconds" %q17_cold)
    print ("The fastest time for the 17th query hot runs was : %s seconds" %q17_min_hot)
    print ("The slowest time for the 17th hot runs was : %s" %q17_max_hot)
    print ("Average time for the 17th query hot runs is : %s seconds" %q17_avg)
    print ("---------------------------------------------------------------")

    run_times18=[]
    for i in range(1,num_times):
            start18=time.time()
            cur.execute("""Select c_name,c_custkey,o_orderkey, o_orderdate, o_totalprice, sum(l_quantity) from customer, orders, lineitem where o_orderkey in (Select l_orderkey from lineitem group by l_orderkey having sum(l_quantity)>300) and c_custkey=o_custkey and o_orderkey=l_orderkey group by c_name, c_custkey, o_orderkey,o_orderdate,o_totalprice order by o_totalprice desc, o_orderdate limit 100;""")
            end18=time.time()
            q18=end18-start18
            run_times18.append(q18)
            print ("Elapsed time for the %d run of the 18th query is %s seconds" %(i,q18))

    q18_cold=run_times18[0]
    q18_min_hot=min(run_times18[1:])
    q18_max_hot=max(run_times18[1:])
    q18_avg=(sum(run_times18[1:]))/(len(run_times18)-1)
    print ("The cold run for the 18th query was: %s seconds" %q18_cold)
    print ("The fastest time for the 18th query hot runs was : %s seconds" %q18_min_hot)
    print ("The slowest time for the 18th hot runs was : %s" %q18_max_hot)
    print ("Average time for the 18th query hot runs is : %s seconds" %q18_avg)
    print ("---------------------------------------------------------------")

    run_times19=[]
    for i in range(1,num_times):
            start19=time.time()
            cur.execute("""Select sum(l_extendedprice * (1-l_discount)) as revenrue from lineitem, part where ( p_partkey=l_partkey and p_brand='Brand#12' and p_container in ('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG') and l_quantity >=1 and l_quantity <= 1+10 and p_size between 1 and 5 and l_shipmode in ('AIR', 'AIR REG') and l_shipinstruct='DELIVER IN PERSON') or ( p_partkey=l_partkey and p_brand ='Brand#23' and p_container in ('MED BAG','MED BOX', 'MED PKG', 'MED PACK') and l_quantity >=10 and l_quantity <= 10+10 and p_size between 1 and 10 and l_shipmode in ('AIR','AIR REG') and l_shipinstruct='DELIVER IN PERSON') or (p_partkey=l_partkey and p_brand ='Brand#34' and p_container in ('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG') and l_quantity >= 20 and l_quantity <=20+10 and p_size between 1 and 15 and l_shipmode in ('AIR', 'AIR REG') and l_shipinstruct='DELIVER IN PERSON');""")
            end19=time.time()
            q19=end19-start19
            run_times19.append(q19)
            print ("Elapsed time for the %d run of the 19th query is %s seconds" %(i,q19))

    q19_cold=run_times19[0]
    q19_min_hot=min(run_times19[1:])
    q19_max_hot=max(run_times19[1:])
    q19_avg=(sum(run_times19[1:]))/(len(run_times19)-1)
    print ("The cold run for the 19th query was: %s seconds" %q19_cold)
    print ("The fastest time for the 19th query hot runs was : %s seconds" %q19_min_hot)
    print ("The slowest time for the 19th hot runs was : %s" %q19_max_hot)
    print ("Average time for the 19th query hot runs is : %s seconds" %q19_avg)
    print ("---------------------------------------------------------------")

    run_times20=[]
    for i in range(1,num_times):
            start20=time.time()
            cur.execute("""Select s_name, s_address from supplier, nation where s_suppkey in ( Select ps_suppkey from partsupp where ps_partkey in (Select p_partkey from part where p_name like 'forest%' ) and ps_availqty > ( Select 0.5 * sum(l_quantity) from lineitem where l_partkey=ps_partkey and l_suppkey = ps_suppkey and l_shipdate >= date '1994-01-01' and l_shipdate < date '1994-01-01' + interval '1' year)) and s_nationkey=n_nationkey and s_name ='CANADA' order by s_name;""")
            end20=time.time()
            q20=end20-start20
            run_times20.append(q20)
            print ("Elapsed time for the %d run of the 20th query is %s seconds" %(i,q20))

    q20_cold=run_times20[0]
    q20_min_hot=min(run_times20[1:])
    q20_max_hot=max(run_times20[1:])
    q20_avg=(sum(run_times20[1:]))/(len(run_times20)-1)
    print ("The cold run for the 20th query was: %s seconds" %q20_cold)
    print ("The fastest time for the 20th query hot runs was : %s seconds" %q20_min_hot)
    print ("The slowest time for the 20th hot runs was : %s" %q20_max_hot)
    print ("Average time for the 20th query hot runs is : %s seconds" %q20_avg)
    print ("---------------------------------------------------------------")
    
    run_times21=[]
    for i in range(1, num_times):
            start21=time.time()
            cur.execute("""Select s_name, count(*) as numwait from supplier, lineitem l1, orders, nation where s_suppkey=l1.l_suppkey and o_orderkey=l1.l_orderkey and o_orderstatus='F' and l1.l_receiptdate > l1.l_commitdate and exists ( Select * from lineitem l2 where l2.l_orderkey=l1.l_orderkey and l2.l_suppkey <> l1.l_suppkey) and not exists ( Select * from lineitem l3 where l3.l_orderkey =l1.l_orderkey and l3.l_suppkey<> l1.l_suppkey and l3.l_receiptdate > l3.l_commitdate) and s_nationkey=n_nationkey and n_name='SAUDI ARABIA' group by s_name order by numwait desc, s_name limit 100;""")
            end21=time.time()
            q21=end21-start21
            run_times21.append(q21)
            print ("Elapsed time for the %d run of the 21st query is %s seconds" %(i,q21))

    q21_cold=run_times21[0]
    q21_min_hot=min(run_times21[1:])
    q21_max_hot=max(run_times21[1:])
    q21_avg=(sum(run_times21[1:]))/(len(run_times21)-1)
    print ("The cold run for the 21st query was: %s seconds" %q21_cold)
    print ("The fastest time for the 21st query hot runs was : %s seconds" %q21_min_hot)
    print ("The slowest time for the 21st hot runs was : %s" %q21_max_hot)
    print ("Average time for the 21st query hot runs is : %s seconds" %q21_avg)
    print ("---------------------------------------------------------------")

    run_times22=[]
    for i in range(1, num_times):
            start22=time.time()
            cur.execute("""Select cntrycode, count(*) as numcust, sum(c_acctbal) as totacctbal from ( Select substring(c_phone from 1 for 2) as cntrycode,c_acctbal from customer where substring(c_phone from 1 for 2) in ('13','31','23','29','30','18','17') and c_acctbal > ( Select avg(c_acctbal) from customer where c_acctbal >0.00 and substring (c_phone from 1 for 2) in ('13','31','23','29','30','18','17')) and not exists ( Select * from orders where o_custkey=c_custkey)) as custsale group by cntrycode order by cntrycode;""")
            end22=time.time()
            q22=end22-start22
            run_times22.append(q22)
            print ("Elapsed time for the %d run of the 22nd query is %s seconds" %(i,q22))

    q22_cold=run_times22[0]
    q22_min_hot=min(run_times22[1:])
    q22_max_hot=max(run_times22[1:])
    q22_avg=(sum(run_times22[1:]))/(len(run_times22)-1)
    print ("The cold run for the 22nd query was: %s seconds" %q22_cold)
    print ("The fastest time for the 22nd query hot runs was : %s seconds" %q22_min_hot)
    print ("The slowest time for the 22nd hot runs was : %s" %q22_max_hot)
    print ("Average time for the 22nd query hot runs is : %s seconds" %q22_avg)
    print ("---------------------------------------------------------------")

#---------------------UPDATE TRY---------------------------------

    os.chdir(pathupdatecommit)
    print "Current commit version is : "
    ComValue1=os.system("git rev-parse HEAD")
    print "Updating to the new commit of Postgresql.."
    os.system("git pull https://github.com/postgres/postgres")
    print "--------------------------------------------------"
    print "Current commit version is : "
    ComValue2=os.system("git rev-parse HEAD")
    #while ComValue1==ComValue2:
        #print ("There is no new commit available, waiting for ? mins..")
        #os.system("sleep 3m")
        #os.system("git pull https://github.com/postgres/postgres")
        #ComValue2=os.system("git rev-parse")
    pathpostupdate=os.path.join(home,Dirvar,'bin')
    os.chdir(pathpostupdate)

    num_times=6
    run_times1b=[]
    for i in range(1,num_times):
            start1b=time.time()
            cur.execute("""Select l_returnflag,l_linestatus,sum(l_quantity) as sum_qty, sum(l_extendedprice) as sum_base_price,sum(l_extendedprice * (1 - l_discount)) as sum_disc_price,sum(l_extendedprice * (1-l_discount) * (1 + l_tax)) as sum_charge, avg(l_quantity) as avg_qty,avg(l_extendedprice) as avg_price, avg(l_discount) as avg_disc,count(*) as count_order from lineitem where l_shipdate <= date '1998-12-01' - interval '90' day group by l_returnflag,l_linestatus order by l_returnflag,l_linestatus;""")
            end1b=time.time()
            q1b=end1b-start1b
            run_times1b.append(q1b)
            print ("Elapsed time for the %d run of the 1st query is : %s seconds" %(i,q1b))
    
    q1_coldb=run_times1b[0]
    q1_min_hotb=min(run_times1b[1:])
    q1_max_hotb=max(run_times1b[1:])
    q1_avgb=(sum(run_times1b[1:]))/(len(run_times1b)-1)
    print ("The cold run for the 1st query was: %s seconds" %q1_coldb)
    print ("The fastest time for the 1st query hot runs was : %s seconds" %q1_min_hotb)
    print ("The slowest time for the hot runs was : %s" %q1_max_hotb)
    print ("Average time for the 1st query hot runs is : %s seconds" %q1_avgb)
    print ("---------------------------------------------------------------")

    run_times2b=[]
    for i in range(1,num_times):
            start2b=time.time()
            cur.execute("""Select s_acctbal,s_name,n_name,p_partkey,p_mfgr,s_address,s_phone,s_comment from part,supplier,partsupp,nation,region where p_partkey=ps_partkey and s_suppkey=ps_suppkey and p_size=15 and p_type like '%BRASS' and s_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='EUROPE' and ps_supplycost=(Select min(ps_supplycost) from partsupp,supplier,nation,region where p_partkey=ps_partkey and s_suppkey=ps_suppkey and s_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='EUROPE') order by s_acctbal desc, n_name, s_name, p_partkey limit 100;""")
            end2b=time.time()
            q2b=end2b-start2b
            run_times2b.append(q2b)
            print ("Elapsed time for the %d run of the 2nd query is : %s seconds" %(i,q2b))

        
    q2_coldb=run_times2b[0]
    q2_min_hotb=min(run_times2b[1:])
    q2_max_hotb=max(run_times2b[1:])
    q2_avgb=(sum(run_times2b[1:]))/(len(run_times2b)-1)
    print ("The cold run for the 2nd query was: %s seconds" %q2_coldb)
    print ("The fastest time for the 2nd query hot runs was : %s seconds" %q2_min_hotb)
    print ("The slowest time for 2nd query hot runs was : %s" %q2_max_hotb)
    print ("Average time for the 2nd query hot runs is : %s seconds" %q2_avgb)
    print ("---------------------------------------------------------------")
   

    run_times3b=[]
    for i in range(1,num_times):
            start3b=time.time()
            cur.execute("""Select l_orderkey,sum(l_extendedprice * (1-l_discount)) as revenue, o_orderdate,o_shippriority from customer,orders,lineitem where c_mktsegment='BUILDING' and c_custkey=o_custkey and l_orderkey=o_orderkey and o_orderdate<date '1995-03-15' and l_shipdate>date '1995-03-15' group by l_orderkey, o_orderdate, o_shippriority order by revenue desc, o_orderdate limit 10;""")
            end3b=time.time()
            q3b=end3b-start3b
            run_times3b.append(q3b)
            print ("Elapsed time for the %d run of the 3rd query is : %s seconds" %(i, q3b))

    q3_coldb=run_times3b[0]
    q3_min_hotb=min(run_times3b[1:])
    q3_max_hotb=max(run_times3b[1:])
    q3_avgb=(sum(run_times3b[1:]))/(len(run_times3b)-1)
    print ("The cold run for the 3rd query was: %s seconds" %q3_coldb)
    print ("The fastest time for the 3rd query hot runs was : %s seconds" %q3_min_hotb)
    print ("The slowest time for the 3rd query hot runs was : %s" %q3_max_hotb)
    print ("Average time for the 3rd query hot runs is : %s seconds" %q3_avgb)
    print ("---------------------------------------------------------------")
    
    run_times4b=[]
    for i in range(1,num_times):
            start4b=time.time()
            cur.execute("""Select o_orderpriority, count(*) as order_count from orders where o_orderdate>= date '1993-07-01' and o_orderdate < date '1993-07-01' + interval '3' month and exists ( Select * from lineitem where l_orderkey=o_orderkey and l_commitdate < l_receiptdate) group by o_orderpriority order by o_orderpriority;""")
            end4b=time.time()
            q4b=end4b-start4b
            run_times4b.append(q4b)
            print ("Elapsed time for the %d run of the 4th query is : %s seconds" % (i,q4b))

    q4_coldb=run_times4b[0]
    q4_min_hotb=min(run_times4b[1:])
    q4_max_hotb=max(run_times4b[1:])
    q4_avgb=(sum(run_times4b[1:]))/(len(run_times4b)-1)
    print ("The cold run for the 4th query was: %s seconds" %q4_cold)
    print ("The fastest time for the 4th query hot runs was : %s seconds" %q4_min_hot)
    print ("The slowest time for the 4th query runs was : %s" %q4_max_hot)
    print ("Average time for the 4th query hot runs is : %s seconds" %q4_avg)
    print ("---------------------------------------------------------------")

    run_times5b=[]
    for i in range(1, num_times):
            start5b = time.time()
            cur.execute("""Select n_name, sum(l_extendedprice * (1-l_discount)) as revenue from customer, orders, lineitem, supplier, nation, region where c_custkey=o_custkey and l_orderkey=o_orderkey and l_suppkey=s_suppkey and c_nationkey=n_nationkey and n_regionkey=r_regionkey and r_name='ASIA' and o_orderdate >=date '1994-01-01' and o_orderdate>=date '1994-01-01' and o_orderdate<date '1994-01-01' + interval '1' year group by n_name order by revenue desc;""")
            end5b=time.time()
            q5b = end5b - start5b
            run_times5b.append(q5b)
            print ("Elapsed time for the run %d of the 5th query is : %s seconds" % (i, q5b) )
        
    q5_coldb=run_times5b[0]
    q5_min_hotb=min(run_times5b[1:])
    q5_max_hotb=max(run_times5b[1:])
    q5_avgb=(sum(run_times5b[1:]))/(len(run_times5b)-1)
    print ("The cold run for the 5th query was: %s seconds" %q5_coldb)
    print ("The fastest time for the 5th query hot runs was : %s seconds" %q5_min_hotb)
    print ("The slowest time for the 5th hot runs was : %s" %q5_max_hotb)
    print ("Average time for the 5th query hot runs is : %s seconds" %q5_avgb)
    print ("---------------------------------------------------------------")
 
    run_times6b=[]
    for i in range(1,num_times):
            start6b=time.time()
            cur.execute("""Select sum(l_extendedprice * l_discount) as revenue from lineitem where l_shipdate>=date '1994-01-01' and l_shipdate < date '1994-01-01' + interval '1' year and l_discount between 0.06 - 0.01 and 0.06 + 0.01 and l_quantity<24;""")
            end6b=time.time()
            q6b=end6b-start6b
            run_times6b.append(q6b)
            print ("Elapsed time for the %d run of the 6th query is : %s seconds" % (i,q6b))

    q6_coldb=run_times6b[0]
    q6_min_hotb=min(run_times6b[1:])
    q6_max_hotb=max(run_times6b[1:])
    q6_avgb=(sum(run_times6b[1:]))/(len(run_times6b)-1)
    print ("The cold run for the 6th query was: %s seconds" %q6_coldb)
    print ("The fastest time for the 6th query hot runs was : %s seconds" %q6_min_hotb)
    print ("The slowest time for the 6th hot runs was : %s" %q6_max_hotb)
    print ("Average time for the 6th query hot runs is : %s seconds" %q6_avgb)
    print ("---------------------------------------------------------------")

    run_times7b=[]
    for i in range(1,num_times):
            start7b=time.time()
            cur.execute("""Select supp_nation,cust_nation,l_year,sum(volume) as revenue from ( Select n1.n_name as supp_nation, n2.n_name as cust_nation, extract(year from l_shipdate) as l_year, l_extendedprice * (1-l_discount) as volume from supplier,lineitem,orders,customer,nation n1, nation n2 where s_suppkey=l_suppkey and o_orderkey=l_orderkey and c_custkey=o_custkey and s_nationkey=n1.n_nationkey and c_nationkey=n2.n_nationkey and ((n1.n_name='FRANCE' and n2.n_name='GERMANY') or (n1.n_name='GERMANY' and n2.n_name='FRANCE')) and l_shipdate between date '1995-01-01' and date '1996-12-31') as shipping group by supp_nation,cust_nation,l_year order by supp_nation,cust_nation,l_year;""")
            end7b=time.time()
            q7b=end7b-start7b
            run_times7b.append(q7b)
            print ("Elapsed time for the %d run of the 7th query is : %s seconds" %(i, q7b))

    q7_coldb=run_times7b[0]
    q7_min_hotb=min(run_times7b[1:])
    q7_max_hotb=max(run_times7b[1:])
    q7_avgb=(sum(run_times7b[1:]))/(len(run_times7b)-1)
    print ("The cold run for the 7th query was: %s seconds" %q7_coldb)
    print ("The fastest time for the 7th query hot runs was : %s seconds" %q7_min_hotb)
    print ("The slowest time for the 7th hot runs was : %s" %q7_max_hotb)
    print ("Average time for the 7th query hot runs is : %s seconds" %q7_avgb)
    print ("---------------------------------------------------------------")
        
    run_times8b=[]
    for i in range(1,num_times):
            start8b=time.time()
            cur.execute("""Select o_year, sum(case when nation='BRAZIL' then volume else 0 end)/sum(volume) as mkt_share from (Select extract(year from o_orderdate) as o_year,l_extendedprice * (1-l_discount) as volume, n2.n_name as nation from part, supplier, lineitem, orders, customer, nation n1, nation n2, region where p_partkey=l_partkey and s_suppkey=l_suppkey and l_orderkey=o_orderkey and o_custkey=c_custkey and c_nationkey=n1.n_nationkey and n1.n_regionkey=r_regionkey and r_name='AMERICA' and s_nationkey=n2.n_nationkey and o_orderdate between date '1995-01-01' and date '1996-12-31' and p_type='ECONOMY ANODIZED STEEL') as all_nations group by o_year order by o_year;""")
            end8b=time.time()
            q8b=end8b-start8b
            run_times8b.append(q8b)
            print ("Elapsed time for the %d run of the 8th query is : %s seconds" %(i,q8b))

    q8_coldb=run_times8b[0]
    q8_min_hotb=min(run_times8b[1:])
    q8_max_hotb=max(run_times8b[1:])
    q8_avgb=(sum(run_times8b[1:]))/(len(run_times8b)-1)
    print ("The cold run for the 8th query was: %s seconds" %q8_coldb)
    print ("The fastest time for the 8th query hot runs was : %s seconds" %q8_min_hotb)
    print ("The slowest time for the 8th hot runs was : %s" %q8_max_hotb)
    print ("Average time for the 8th query hot runs is : %s seconds" %q8_avgb)
    print ("---------------------------------------------------------------")

    run_times9b=[]
    for i in range(1,num_times):
            start9b=time.time()
            cur.execute("""Select nation,o_year,sum(amount) as sum_profit from ( Select n_name as nation, extract(year from o_orderdate) as o_year, l_extendedprice * (1-l_discount)-ps_supplycost * l_quantity as amount from part, supplier, lineitem, partsupp, orders, nation where s_suppkey=l_suppkey and ps_suppkey=l_suppkey and ps_partkey=l_partkey and p_partkey=l_partkey and o_orderkey=l_orderkey and s_nationkey=n_nationkey and p_name like '%green%') as profit group by nation, o_year order by nation,o_year desc;""")
            end9b=time.time()
            q9b=end9b-start9b
            run_times9b.append(q9b)
            print ("Elapsed time for the %d run of the 9th query is : %s seconds" % (i,q9b))

    q9_coldb=run_times9b[0]
    q9_min_hotb=min(run_times9b[1:])
    q9_max_hotb=max(run_times9b[1:])
    q9_avgb=(sum(run_times9b[1:]))/(len(run_times9b)-1)
    print ("The cold run for the 9th query was: %s seconds" %q9_coldb)
    print ("The fastest time for the 9th query hot runs was : %s seconds" %q9_min_hotb)
    print ("The slowest time for the 9th hot runs was : %s" %q9_max_hotb)
    print ("Average time for the 9th query hot runs is : %s seconds" %q9_avgb)
    print ("---------------------------------------------------------------")
        
    run_times10b=[]
    for i in range(1,num_times):
            start10b=time.time()
            cur.execute("""Select c_custkey, c_name, sum(l_extendedprice * (1-l_discount)) as revenue, c_acctbal, n_name, c_address, c_phone, c_comment from customer, orders, lineitem, nation where c_custkey=o_custkey and l_orderkey=o_orderkey and o_orderdate>=date '1993-10-01' and o_orderdate < date '1993-10-01' + interval '3' month and l_returnflag='R' and c_nationkey=n_nationkey group by c_custkey, c_name, c_acctbal, c_phone, n_name, c_address, c_comment order by revenue desc limit 20;""")
            end10b=time.time()
            q10b=end10b-start10b
            run_times10b.append(q10b)
            print ("Elapsed time for the %d run of the 10th query is : %s seconds" %(i,q10b))

    q10_coldb=run_times10b[0]
    q10_min_hotb=min(run_times10b[1:])
    q10_max_hotb=max(run_times10b[1:])
    q10_avgb=(sum(run_times10b[1:]))/(len(run_times10b)-1)
    print ("The cold run for the 10th query was: %s seconds" %q10_coldb)
    print ("The fastest time for the 10th query hot runs was : %s seconds" %q10_min_hotb)
    print ("The slowest time for the 10th hot runs was : %s" %q10_max_hotb)
    print ("Average time for the 10th query hot runs is : %s seconds" %q10_avgb)
    print ("---------------------------------------------------------------")

    run_times11b=[]
    for i in range(1,num_times):
            start11b=time.time()
            cur.execute("""Select ps_partkey, sum(ps_supplycost * ps_availqty) as value from partsupp, supplier, nation where ps_suppkey=s_suppkey and s_nationkey=n_nationkey and n_name='GERMANY' group by ps_partkey having sum(ps_supplycost * ps_availqty)>(Select sum(ps_supplycost * ps_availqty) * 0.0001000000 from partsupp, supplier, nation where ps_suppkey=s_suppkey and s_nationkey=n_nationkey and n_name='GERMANY') order by value desc;""")
            end11b=time.time()
            q11b=end11b-start11b
            run_times11b.append(q11b)
            print ("Elapsed time for the %d run of the 11th query is : %s seconds" %(i, q11b))

    q11_coldb=run_times11b[0]
    q11_min_hotb=min(run_times11b[1:])
    q11_max_hotb=max(run_times11b[1:])
    q11_avgb=(sum(run_times11b[1:]))/(len(run_times11b)-1)
    print ("The cold run for the 11th query was: %s seconds" %q11_coldb)
    print ("The fastest time for the 11th query hot runs was : %s seconds" %q11_min_hotb)
    print ("The slowest time for the 11th hot runs was : %s" %q11_max_hotb)
    print ("Average time for the 11th query hot runs is : %s seconds" %q11_avgb)
    print ("---------------------------------------------------------------")

    run_times12b=[]
    for i in range(1,num_times):
            start12b=time.time()
            cur.execute("""Select l_shipmode,sum(case when o_orderpriority='1-URGENT' or o_orderpriority='2-HIGH' then 1 else 0 end) as high_line_count, sum(case when o_orderpriority <> '1=URGENT' and o_orderpriority <> '2-HIGH' then 1 else 0 end) as low_line_count from orders,lineitem where o_orderkey=l_orderkey and l_shipmode in('MAIL','SHIP') and l_commitdate < l_receiptdate and l_shipdate < l_commitdate and l_receiptdate >= date '1994-01-01' and l_receiptdate < date '1004-01-01'+ interval '1' year group by l_shipmode order by l_shipmode;""")
            end12b=time.time()
            q12b=end12b-start12b
            run_times12b.append(q12b)
            print ("Elapsed time for the %d run of the 12th query is : %s seconds" %(i,q12b))

    q12_coldb=run_times12b[0]
    q12_min_hotb=min(run_times12b[1:])
    q12_max_hotb=max(run_times12b[1:])
    q12_avgb=(sum(run_times12b[1:]))/(len(run_times12b)-1)
    print ("The cold run for the 12th query was: %s seconds" %q12_coldb)
    print ("The fastest time for the 12th query hot runs was : %s seconds" %q12_min_hotb)
    print ("The slowest time for the 12th hot runs was : %s" %q12_max_hotb)
    print ("Average time for the 12th query hot runs is : %s seconds" %q12_avgb)
    print ("---------------------------------------------------------------")

    run_times13b=[]
    for i in range(1,num_times):
            start13b=time.time()
            cur.execute("""Select c_count,count(*) as custdist from ( Select c_custkey, count(o_orderkey) from customer left outer join orders on c_custkey=o_custkey and o_comment not like '%special%requests%' group by c_custkey) as c_orders(c_custkey,c_count) group by c_count order by custdist desc, c_count desc;""")
            end13b=time.time()
            q13b=end13b-start13b
            run_times13b.append(q13b)
            print ("Elapsed time for the %d run of the 13th query is %s seconds" %(i,q13b))

    q13_coldb=run_times13b[0]
    q13_min_hotb=min(run_times13b[1:])
    q13_max_hotb=max(run_times13b[1:])
    q13_avgb=(sum(run_times13b[1:]))/(len(run_times13b)-1)
    print ("The cold run for the 13th query was: %s seconds" %q13_coldb)
    print ("The fastest time for the 13th query hot runs was : %s seconds" %q13_min_hotb)
    print ("The slowest time for the 13th hot runs was : %s" %q13_max_hotb)
    print ("Average time for the 13th query hot runs is : %s seconds" %q13_avgb)
    print ("---------------------------------------------------------------")

    run_times14b=[]
    for i in range(1,num_times):
            start14b=time.time()
            cur.execute("""Select 100.00 * sum(case when p_type like 'PROMO%' then l_extendedprice * (1-l_discount) else 0 end) / sum(l_extendedprice * (1-l_discount)) as promo_revenue from lineitem,part where l_partkey=p_partkey and l_shipdate >= date '1995-09-01' and l_shipdate < date '1995-09-01' + interval '1' month;""")
            end14b=time.time()
            q14b=end14b-start14b
            run_times14b.append(q14b)
            print ("Elapsed time for the %d run of the 14th query is %s seconds" %(i,q14b))

    q14_coldb=run_times14b[0]
    q14_min_hotb=min(run_times14b[1:])
    q14_max_hotb=max(run_times14b[1:])
    q14_avgb=(sum(run_times14b[1:]))/(len(run_times14b)-1)
    print ("The cold run for the 14th query was: %s seconds" %q14_coldb)
    print ("The fastest time for the 14th query hot runs was : %s seconds" %q14_min_hotb)
    print ("The slowest time for the 14th hot runs was : %s" %q14_max_hotb)
    print ("Average time for the 14th query hot runs is : %s seconds" %q14_avgb)
    print ("---------------------------------------------------------------")
    
    run_times15b=[]
    for i in range(1,num_times):
            start15b=time.time()
            cur.execute("""Create view revenue0(supplier_no, total_revenue) as select l_suppkey, sum(l_extendedprice * (1-l_discount)) from lineitem where l_shipdate >= date '1996-01-01' and l_shipdate < date '1996-01-01' + interval '3' month group by l_suppkey; Select s_suppkey,s_name, s_address, s_phone, total_revenue from supplier, revenue0 where s_suppkey=supplier_no and total_revenue=(select max(total_revenue) from revenue0) order by s_suppkey; drop view revenue0;""")
            end15b=time.time()
            q15b=end15b-start15b
            run_times15b.append(q15b)
            print ("Elapsed time for the %d run of the 15th query is %s seconds" %(i,q15b))

    q15_coldb=run_times15b[0]
    q15_min_hotb=min(run_times15b[1:])
    q15_max_hotb=max(run_times15b[1:])
    q15_avgb=(sum(run_times15b[1:]))/(len(run_times15b)-1)
    print ("The cold run for the 15th query was: %s seconds" %q15_coldb)
    print ("The fastest time for the 15th query hot runs was : %s seconds" %q15_min_hotb)
    print ("The slowest time for the 15th hot runs was : %s" %q15_max_hotb)
    print ("Average time for the 15th query hot runs is : %s seconds" %q15_avgb)
    print ("---------------------------------------------------------------")

    run_times16b=[]
    for i in range(1,num_times):
            start16b=time.time()
            cur.execute("""Select p_brand, p_type, p_size, count(distinct ps_suppkey) as supplier_cnt from partsupp, part where p_partkey=ps_partkey and p_brand <> 'Brand#45'and p_type not like 'MEDIUM POLISHED%' and p_size in (49,14,23,45,19,3,36,9) and ps_suppkey not in( Select s_suppkey from supplier where s_comment like '%Customer%Complaints%') group by p_brand,p_type,p_size order by supplier_cnt desc, p_brand,p_type, p_size;""")
            end16b=time.time()
            q16b=end16b-start16b
            run_times16b.append(q16b)
            print ("Elapsed time for the %d run of the 16th query is %s seconds" %(i,q16b))

    q16_coldb=run_times16b[0]
    q16_min_hotb=min(run_times16b[1:])
    q16_max_hotb=max(run_times16b[1:])
    q16_avgb=(sum(run_times16b[1:]))/(len(run_times16b)-1)
    print ("The cold run for the 16th query was: %s seconds" %q16_coldb)
    print ("The fastest time for the 16th query hot runs was : %s seconds" %q16_min_hotb)
    print ("The slowest time for the 16th hot runs was : %s" %q16_max_hotb)
    print ("Average time for the 16th query hot runs is : %s seconds" %q16_avgb)
    print ("---------------------------------------------------------------")

    run_times17b=[]
    for i in range(1,num_times):
            start17b=time.time()
            cur.execute("""Select sum(l_extendedprice) / 7.0 as avg_yearly from lineitem, part where p_partkey=l_partkey and p_brand='Brand#23' and p_container='MED BOX' and l_quantity < (Select 0.2 * avg(l_quantity) from lineitem where l_partkey=p_partkey);""")
            end17b=time.time()
            q17b=end17b-start17b
            run_times17b.append(q17b)
            print ("Elapsed time for the %d run of the 17th query is %s seconds" %(i,q17b))

    q17_coldb=run_times17b[0]
    q17_min_hotb=min(run_times17b[1:])
    q17_max_hotb=max(run_times17b[1:])
    q17_avgb=(sum(run_times17b[1:]))/(len(run_times17b)-1)
    print ("The cold run for the 17th query was: %s seconds" %q17_coldb)
    print ("The fastest time for the 17th query hot runs was : %s seconds" %q17_min_hotb)
    print ("The slowest time for the 17th hot runs was : %s" %q17_max_hotb)
    print ("Average time for the 17th query hot runs is : %s seconds" %q17_avgb)
    print ("---------------------------------------------------------------")

    run_times18b=[]
    for i in range(1,num_times):
            start18b=time.time()
            cur.execute("""Select c_name,c_custkey,o_orderkey, o_orderdate, o_totalprice, sum(l_quantity) from customer, orders, lineitem where o_orderkey in (Select l_orderkey from lineitem group by l_orderkey having sum(l_quantity)>300) and c_custkey=o_custkey and o_orderkey=l_orderkey group by c_name, c_custkey, o_orderkey,o_orderdate,o_totalprice order by o_totalprice desc, o_orderdate limit 100;""")
            end18b=time.time()
            q18b=end18b-start18b
            run_times18b.append(q18b)
            print ("Elapsed time for the %d run of the 18th query is %s seconds" %(i,q18b))

    q18_coldb=run_times18b[0]
    q18_min_hotb=min(run_times18b[1:])
    q18_max_hotb=max(run_times18b[1:])
    q18_avgb=(sum(run_times18b[1:]))/(len(run_times18b)-1)
    print ("The cold run for the 18th query was: %s seconds" %q18_coldb)
    print ("The fastest time for the 18th query hot runs was : %s seconds" %q18_min_hotb)
    print ("The slowest time for the 18th hot runs was : %s" %q18_max_hotb)
    print ("Average time for the 18th query hot runs is : %s seconds" %q18_avgb)
    print ("---------------------------------------------------------------")

    run_times19b=[]
    for i in range(1,num_times):
            start19b=time.time()
            cur.execute("""Select sum(l_extendedprice * (1-l_discount)) as revenrue from lineitem, part where ( p_partkey=l_partkey and p_brand='Brand#12' and p_container in ('SM CASE', 'SM BOX', 'SM PACK', 'SM PKG') and l_quantity >=1 and l_quantity <= 1+10 and p_size between 1 and 5 and l_shipmode in ('AIR', 'AIR REG') and l_shipinstruct='DELIVER IN PERSON') or ( p_partkey=l_partkey and p_brand ='Brand#23' and p_container in ('MED BAG','MED BOX', 'MED PKG', 'MED PACK') and l_quantity >=10 and l_quantity <= 10+10 and p_size between 1 and 10 and l_shipmode in ('AIR','AIR REG') and l_shipinstruct='DELIVER IN PERSON') or (p_partkey=l_partkey and p_brand ='Brand#34' and p_container in ('LG CASE', 'LG BOX', 'LG PACK', 'LG PKG') and l_quantity >= 20 and l_quantity <=20+10 and p_size between 1 and 15 and l_shipmode in ('AIR', 'AIR REG') and l_shipinstruct='DELIVER IN PERSON');""")
            end19b=time.time()
            q19b=end19b-start19b
            run_times19b.append(q19b)
            print ("Elapsed time for the %d run of the 19th query is %s seconds" %(i,q19b))

    q19_coldb=run_times19b[0]
    q19_min_hotb=min(run_times19b[1:])
    q19_max_hotb=max(run_times19b[1:])
    q19_avgb=(sum(run_times19b[1:]))/(len(run_times19b)-1)
    print ("The cold run for the 19th query was: %s seconds" %q19_coldb)
    print ("The fastest time for the 19th query hot runs was : %s seconds" %q19_min_hotb)
    print ("The slowest time for the 19th hot runs was : %s" %q19_max_hotb)
    print ("Average time for the 19th query hot runs is : %s seconds" %q19_avgb)
    print ("---------------------------------------------------------------")

    run_times20b=[]
    for i in range(1,num_times):
            start20b=time.time()
            cur.execute("""Select s_name, s_address from supplier, nation where s_suppkey in ( Select ps_suppkey from partsupp where ps_partkey in (Select p_partkey from part where p_name like 'forest%' ) and ps_availqty > ( Select 0.5 * sum(l_quantity) from lineitem where l_partkey=ps_partkey and l_suppkey = ps_suppkey and l_shipdate >= date '1994-01-01' and l_shipdate < date '1994-01-01' + interval '1' year)) and s_nationkey=n_nationkey and s_name ='CANADA' order by s_name;""")
            end20b=time.time()
            q20b=end20b-start20b
            run_times20b.append(q20b)
            print ("Elapsed time for the %d run of the 20th query is %s seconds" %(i,q20b))

    q20_coldb=run_times20b[0]
    q20_min_hotb=min(run_times20b[1:])
    q20_max_hotb=max(run_times20b[1:])
    q20_avgb=(sum(run_times20b[1:]))/(len(run_times20b)-1)
    print ("The cold run for the 20th query was: %s seconds" %q20_coldb)
    print ("The fastest time for the 20th query hot runs was : %s seconds" %q20_min_hotb)
    print ("The slowest time for the 20th hot runs was : %s" %q20_max_hotb)
    print ("Average time for the 20th query hot runs is : %s seconds" %q20_avgb)
    print ("---------------------------------------------------------------")

    run_times21b=[]
    for i in range(1, num_times):
            start21b=time.time()
            cur.execute("""Select s_name, count(*) as numwait from supplier, lineitem l1, orders, nation where s_suppkey=l1.l_suppkey and o_orderkey=l1.l_orderkey and o_orderstatus='F' and l1.l_receiptdate > l1.l_commitdate and exists ( Select * from lineitem l2 where l2.l_orderkey=l1.l_orderkey and l2.l_suppkey <> l1.l_suppkey) and not exists ( Select * from lineitem l3 where l3.l_orderkey =l1.l_orderkey and l3.l_suppkey<> l1.l_suppkey and l3.l_receiptdate > l3.l_commitdate) and s_nationkey=n_nationkey and n_name='SAUDI ARABIA' group by s_name order by numwait desc, s_name limit 100;""")
            end21b=time.time()
            q21b=end21b-start21b
            run_times21b.append(q21b)
            print ("Elapsed time for the %d run of the 21st query is %s seconds" %(i,q21b))

    q21_coldb=run_times21b[0]
    q21_min_hotb=min(run_times21b[1:])
    q21_max_hotb=max(run_times21b[1:])
    q21_avgb=(sum(run_times21b[1:]))/(len(run_times21b)-1)
    print ("The cold run for the 21st query was: %s seconds" %q21_coldb)
    print ("The fastest time for the 21st query hot runs was : %s seconds" %q21_min_hotb)
    print ("The slowest time for the 21st hot runs was : %s" %q21_max_hotb)
    print ("Average time for the 21st query hot runs is : %s seconds" %q21_avgb)
    print ("---------------------------------------------------------------")

    run_times22b=[]
    for i in range(1, num_times):
            start22b=time.time()
            cur.execute("""Select cntrycode, count(*) as numcust, sum(c_acctbal) as totacctbal from ( Select substring(c_phone from 1 for 2) as cntrycode,c_acctbal from customer where substring(c_phone from 1 for 2) in ('13','31','23','29','30','18','17') and c_acctbal > ( Select avg(c_acctbal) from customer where c_acctbal >0.00 and substring (c_phone from 1 for 2) in ('13','31','23','29','30','18','17')) and not exists ( Select * from orders where o_custkey=c_custkey)) as custsale group by cntrycode order by cntrycode;""")
            end22b=time.time()
            q22b=end22b-start22b
            run_times22b.append(q22b)
            print ("Elapsed time for the %d run of the 22nd query is %s seconds" %(i,q22b))

    q22_coldb=run_times22b[0]
    q22_min_hotb=min(run_times22b[1:])
    q22_max_hotb=max(run_times22b[1:])
    q22_avgb=(sum(run_times22b[1:]))/(len(run_times22b)-1)
    print ("The cold run for the 22nd query was: %s seconds" %q22_coldb)
    print ("The fastest time for the 22nd query hot runs was : %s seconds" %q22_min_hotb)
    print ("The slowest time for the 22nd hot runs was : %s" %q22_max_hotb)
    print ("Average time for the 22nd query hot runs is : %s seconds" %q22_avgb)
    print ("---------------------------------------------------------------")

    ListBeforeUpd=[q1_cold,q1_min_hot,q1_max_hot,q1_avg,q2_cold,q2_min_hot,q2_max_hot,q2_avg,q3_cold,q3_min_hot,q3_max_hot,q3_avg,q4_cold,q4_min_hot,q4_max_hot,q4_avg,q5_cold,q5_min_hot,q5_max_hot,q5_avg,q6_cold,q6_min_hot,q6_max_hot,q6_avg,q7_cold,q7_min_hot,q7_max_hot,q7_avg,q8_cold,q8_min_hot,q8_max_hot,q8_avg,q9_cold,q9_min_hot,q9_max_hot,q9_avg,q10_cold,q10_min_hot,q10_max_hot,q10_avg,q11_cold,q11_min_hot,q11_max_hot,q11_avg,q12_cold,q12_min_hot,q12_max_hot,q12_avg,q13_cold,q13_min_hot,q13_max_hot,q13_avg,q14_cold,q14_min_hot,q14_max_hot,q14_avg,q15_cold,q15_min_hot,q15_max_hot,q15_avg,q16_cold,q16_min_hot,q16_max_hot,q16_avg,q17_cold,q17_min_hot,q17_max_hot,q17_avg,q18_cold,q18_min_hot,q18_max_hot,q18_avg,q19_cold,q19_min_hot,q19_max_hot,q19_avg,q20_cold,q20_min_hot,q20_max_hot,q20_avg,q21_cold,q21_min_hot,q21_max_hot,q21_avg,q22_cold,q22_min_hot,q22_max_hot,q22_avg]

    ListAfterUpd=[q1_coldb,q1_min_hotb,q1_max_hotb,q1_avgb,q2_coldb,q2_min_hotb,q2_max_hotb,q2_avgb,q3_coldb,q3_min_hotb,q3_max_hotb,q3_avgb,q4_coldb,q4_min_hotb,q4_max_hotb,q4_avgb,q5_coldb,q5_min_hotb,q5_max_hotb,q5_avgb,q6_coldb,q6_min_hotb,q6_max_hotb,q6_avgb,q7_coldb,q7_min_hotb,q7_max_hotb,q7_avgb,q8_coldb,q8_min_hotb,q8_max_hotb,q8_avgb,q9_coldb,q9_min_hotb,q9_max_hotb,q9_avgb,q10_coldb,q10_min_hotb,q10_max_hotb,q10_avgb,q11_coldb,q11_min_hotb,q11_max_hotb,q11_avgb,q12_coldb,q12_min_hotb,q12_max_hotb,q12_avgb,q13_coldb,q13_min_hotb,q13_max_hotb,q13_avgb,q14_coldb,q14_min_hotb,q14_max_hotb,q14_avgb,q15_coldb,q15_min_hotb,q15_max_hotb,q15_avgb,q16_coldb,q16_min_hotb,q16_max_hotb,q16_avgb,q17_coldb,q17_min_hotb,q17_max_hotb,q17_avgb,q18_coldb,q18_min_hotb,q18_max_hotb,q18_avgb,q19_coldb,q19_min_hotb,q19_max_hotb,q19_avgb,q20_coldb,q20_min_hotb,q20_max_hotb,q20_avgb,q21_coldb,q21_min_hotb,q21_max_hotb,q21_avgb,q22_coldb,q22_min_hotb,q22_max_hotb,q22_avgb]
    
    
    listfin=map(sub,ListAfterUpd,ListBeforeUpd)
    for i in xrange(0, len(listfin), 4):
        print "Results of the %d query for the cold run difference : %s , minimum hot run difference : %s , Maximum hot difference : %s , Average hot runs difference : %s" %((i/4)+1, listfin[i], listfin[i + 1], listfin[i + 2], listfin[i + 3])



objects1=('q1 cold pre update','q1 cold after update','q1 min hot pre update','q1 min hot after update','q1 max hot pre update','q1 max hot after update','q1 avg pre update','q1 avg after update')
y_pos1=np.arange(len(objects1))
x_pos1=[q1_cold,q1_coldb,q1_min_hot,q1_min_hotb,q1_max_hot,q1_max_hotb,q1_avg,q1_avgb]
plt.bar(y_pos1, x_pos1, align='center', alpha=1)
plt.xticks(y_pos1,objects1)
plt.title("Query 1 performance")
plt.ylabel("Seconds")
plt.show()


objects2=('q2 cold pre update','q2 cold after update','q2 min hot pre update','q2 min hot after update','q2 max hot pre update','q2 max hot after update','q2 avg pre update','q2 avg after update')
y_pos2=np.arange(len(objects2))
x_pos2=[q2_cold,q2_coldb,q2_min_hot,q2_min_hotb,q2_max_hot,q2_max_hotb,q2_avg,q2_avgb]
plt.bar(y_pos2, x_pos2, align='center', alpha=1)
plt.xticks(y_pos2,objects2)
plt.title("Query 2 performance")
plt.ylabel("Seconds")
plt.show()

objects3=('q3 cold pre update','q3 cold after update','q3 min hot pre update','q3 min hot after update','q3 max hot pre update','q3 max hot after update','q3 avg pre update','q3 avg after update')
y_pos3=np.arange(len(objects3))
x_pos3=[q3_cold,q3_coldb,q3_min_hot,q3_min_hotb,q3_max_hot,q3_max_hotb,q3_avg,q3_avgb]
plt.bar(y_pos3, x_pos3, align='center', alpha=1)
plt.xticks(y_pos3,objects3)
plt.title("Query 3 performance")
plt.ylabel("Seconds")
plt.show()


objects4=('q4 cold pre update','q4 cold after update','q4 min hot pre update','q4 min hot after update','q4 max hot pre update','q4 max hot after update','q4 avg pre update','q4 avg after update')
y_pos4=np.arange(len(objects4))
x_pos4=[q4_cold,q4_coldb,q4_min_hot,q4_min_hotb,q4_max_hot,q4_max_hotb,q4_avg,q4_avgb]
plt.bar(y_pos4, x_pos4, align='center', alpha=1)
plt.xticks(y_pos4,objects4)
plt.title("Query 4 performance")
plt.ylabel("Seconds")
plt.show()


objects5=('q5 cold pre update','q5 cold after update','q5 min hot pre update','q5 min hot after update','q5 max hot pre update','q5 max hot after update','q5 avg pre update','q5 avg after update')
y_pos5=np.arange(len(objects5))
x_pos5=[q5_cold,q5_coldb,q5_min_hot,q5_min_hotb,q5_max_hot,q5_max_hotb,q5_avg,q5_avgb]
plt.bar(y_pos5, x_pos5, align='center', alpha=1)
plt.xticks(y_pos5,objects5)
plt.title("Query 5 performance")
plt.ylabel("Seconds")
plt.show()

objects6=('q6 cold pre update','q6 cold after update','q6 min hot pre update','q6 min hot after update','q6 max hot pre update','q6 max hot after update','q6 avg pre update','q6 avg after update')
y_pos6=np.arange(len(objects6))
x_pos6=[q6_cold,q6_coldb,q6_min_hot,q6_min_hotb,q6_max_hot,q6_max_hotb,q6_avg,q6_avgb]
plt.bar(y_pos6, x_pos6, align='center', alpha=1)
plt.xticks(y_pos6,objects6)
plt.title("Query 6 performance")
plt.ylabel("Seconds")
plt.show()

objects7=('q7 cold pre update','q7 cold after update','q7 min hot pre update','q7 min hot after update','q7 max hot pre update','q7 max hot after update','q7 avg pre update','q7 avg after update')
y_pos7=np.arange(len(objects7))
x_pos7=[q7_cold,q7_coldb,q7_min_hot,q7_min_hotb,q7_max_hot,q7_max_hotb,q7_avg,q7_avgb]
plt.bar(y_pos7, x_pos7, align='center', alpha=1)
plt.xticks(y_pos7,objects7)
plt.title("Query 7 performance")
plt.ylabel("Seconds")
plt.show()

objects8=('q8 cold pre update','q8 cold after update','q8 min hot pre update','q8 min hot after update','q8 max hot pre update','q8 max hot after update','q8 avg pre update','q8 avg after update')
y_pos8=np.arange(len(objects8))
x_pos8=[q8_cold,q8_coldb,q8_min_hot,q8_min_hotb,q8_max_hot,q8_max_hotb,q8_avg,q8_avgb]
plt.bar(y_pos8, x_pos8, align='center', alpha=1)
plt.xticks(y_pos8,objects8)
plt.title("Query 8 performance")
plt.ylabel("Seconds")
plt.show()

objects9=('q9 cold pre update','q9 cold after update','q9 min hot pre update','q9 min hot after update','q9 max hot pre update','q9 max hot after update','q9 avg pre update','q9 avg after update')
y_pos9=np.arange(len(objects9))
x_pos9=[q9_cold,q9_coldb,q9_min_hot,q9_min_hotb,q9_max_hot,q9_max_hotb,q9_avg,q9_avgb]
plt.bar(y_pos9, x_pos9, align='center', alpha=1)
plt.xticks(y_pos9,objects9)
plt.title("Query 9 performance")
plt.ylabel("Seconds")
plt.show()

objects10=('q10 cold pre update','q10 cold after update','q10 min hot pre update','q10 min hot after update','q10 max hot pre update','q10 max hot after update','q10 avg pre update','q10 avg after update')
y_pos10=np.arange(len(objects10))
x_pos10=[q10_cold,q10_coldb,q10_min_hot,q10_min_hotb,q10_max_hot,q10_max_hotb,q10_avg,q10_avgb]
plt.bar(y_pos10, x_pos10, align='center', alpha=1)
plt.xticks(y_pos10,objects10)
plt.title("Query 10 performance")
plt.ylabel("Seconds")
plt.show()


objects11=('q11 cold pre update','q11 cold after update','q11 min hot pre update','q11 min hot after update','q11 max hot pre update','q11 max hot after update','q11 avg pre update','q11 avg after update')
y_pos11=np.arange(len(objects11))
x_pos11=[q11_cold,q11_coldb,q11_min_hot,q11_min_hotb,q11_max_hot,q11_max_hotb,q11_avg,q11_avgb]
plt.bar(y_pos10, x_pos10, align='center', alpha=1)
plt.xticks(y_pos11,objects11)
plt.title("Query 11 performance")
plt.ylabel("Seconds")
plt.show()


objects12=('q12 cold pre update','q12 cold after update','q12 min hot pre update','q12 min hot after update','q12 max hot pre update','q12 max hot after update','q12 avg pre update','q12 avg after update')
y_pos12=np.arange(len(objects1))
x_pos12=[q12_cold,q12_coldb,q12_min_hot,q12_min_hotb,q12_max_hot,q12_max_hotb,q12_avg,q12_avgb]
plt.bar(y_pos12, x_pos12, align='center', alpha=1)
plt.xticks(y_pos12,objects12)
plt.title("Query 12 performance")
plt.ylabel("Seconds")
plt.show()


objects13=('q13 cold pre update','q13 cold after update','q13 min hot pre update','q13 min hot after update','q13 max hot pre update','q13 max hot after update','q13 avg pre update','q13 avg after update')
y_pos13=np.arange(len(objects1))
x_pos13=[q13_cold,q13_coldb,q13_min_hot,q13_min_hotb,q13_max_hot,q13_max_hotb,q13_avg,q13_avgb]
plt.bar(y_pos13, x_pos13, align='center', alpha=1)
plt.xticks(y_pos13,objects13)
plt.title("Query 13 performance")
plt.ylabel("Seconds")
plt.show()


objects14=('q14 cold pre update','q14 cold after update','q14 min hot pre update','q14 min hot after update','q14 max hot pre update','q14 max hot after update','q14 avg pre update','q14 avg after update')
y_pos14=np.arange(len(objects14))
x_pos14=[q14_cold,q14_coldb,q14_min_hot,q14_min_hotb,q14_max_hot,q14_max_hotb,q14_avg,q14_avgb]
plt.bar(y_pos14, x_pos14, align='center', alpha=1)
plt.xticks(y_pos14,objects14)
plt.title("Query 14 performance")
plt.ylabel("Seconds")
plt.show()


objects15=('q15 cold pre update','q15 cold after update','q15 min hot pre update','q15 min hot after update','q15 max hot pre update','q15 max hot after update','q15 avg pre update','q15 avg after update')
y_pos15=np.arange(len(objects1))
x_pos15=[q15_cold,q15_coldb,q15_min_hot,q15_min_hotb,q15_max_hot,q15_max_hotb,q15_avg,q15_avgb]
plt.bar(y_pos15, x_pos15, align='center', alpha=1)
plt.xticks(y_pos15,objects15)
plt.title("Query 15 performance")
plt.ylabel("Seconds")
plt.show()


objects16=('q16 cold pre update','q16 cold after update','q16 min hot pre update','q16 min hot after update','q16 max hot pre update','q16 max hot after update','q16 avg pre update','q16 avg after update')
y_pos16=np.arange(len(objects1))
x_pos16=[q16_cold,q16_coldb,q16_min_hot,q16_min_hotb,q16_max_hot,q16_max_hotb,q16_avg,q16_avgb]
plt.bar(y_pos16, x_pos16, align='center', alpha=1)
plt.xticks(y_pos16,objects16)
plt.title("Query 16 performance")
plt.ylabel("Seconds")
plt.show()

objects17=('q17 cold pre update','q17 cold after update','q17 min hot pre update','q17 min hot after update','q17 max hot pre update','q17 max hot after update','q17 avg pre update','q17 avg after update')
y_pos17=np.arange(len(objects17))
x_pos17=[q17_cold,q17_coldb,q17_min_hot,q17_min_hotb,q17_max_hot,q17_max_hotb,q17_avg,q17_avgb]
plt.bar(y_pos17, x_pos17, align='center', alpha=1)
plt.xticks(y_pos17,objects17)
plt.title("Query 17 performance")
plt.ylabel("Seconds")
plt.show()

objects18=('q18 cold pre update','q18 cold after update','q18 min hot pre update','q18 min hot after update','q18 max hot pre update','q18 max hot after update','q18 avg pre update','q18 avg after update')
y_pos18=np.arange(len(objects1))
x_pos18=[q18_cold,q18_coldb,q18_min_hot,q18_min_hotb,q18_max_hot,q18_max_hotb,q18_avg,q18_avgb]
plt.bar(y_pos18, x_pos18, align='center', alpha=1)
plt.xticks(y_pos18,objects18)
plt.title("Query 18 performance")
plt.ylabel("Seconds")
plt.show()

objects19=('q19 cold pre update','q19 cold after update','q19 min hot pre update','q19 min hot after update','q19 max hot pre update','q19 max hot after update','q19 avg pre update','q19 avg after update')
y_pos19=np.arange(len(objects19))
x_pos19=[q19_cold,q19_coldb,q19_min_hot,q19_min_hotb,q19_max_hot,q19_max_hotb,q19_avg,q19_avgb]
plt.bar(y_pos19, x_pos19, align='center', alpha=1)
plt.xticks(y_pos19,objects19)
plt.title("Query 19 performance")
plt.ylabel("Seconds")
plt.show()

objects20=('q20 cold pre update','q20 cold after update','q20 min hot pre update','q20 min hot after update','q20 max hot pre update','q20 max hot after update','q20 avg pre update','q20 avg after update')
y_pos20=np.arange(len(objects20))
x_pos20=[q20_cold,q20_coldb,q20_min_hot,q20_min_hotb,q20_max_hot,q20_max_hotb,q20_avg,q20_avgb]
plt.bar(y_pos20, x_pos20, align='center', alpha=1)
plt.xticks(y_pos20,objects20)
plt.title("Query 20 performance")
plt.ylabel("Seconds")
plt.show()

objects21=('q21 cold pre update','q21 cold after update','q21 min hot pre update','q21 min hot after update','q21 max hot pre update','q21 max hot after update','q21 avg pre update','q21 avg after update')
y_pos21=np.arange(len(objects21))
x_pos21=[q21_cold,q21_coldb,q21_min_hot,q21_min_hotb,q21_max_hot,q21_max_hotb,q21_avg,q21_avgb]
plt.bar(y_pos21, x_pos21, align='center', alpha=1)
plt.xticks(y_pos21,objects21)
plt.title("Query 21 performance")
plt.ylabel("Seconds")
plt.show()

objects22=('q22 cold pre update','q22 cold after update','q22 min hot pre update','q22 min hot after update','q22 max hot pre update','q22 max hot after update','q22 avg pre update','q22 avg after update')
y_pos22=np.arange(len(objects22))
x_pos22=[q22_cold,q22_coldb,q22_min_hot,q22_min_hotb,q22_max_hot,q22_max_hotb,q22_avg,q22_avgb]
plt.bar(y_pos22, x_pos22, align='center', alpha=1)
plt.xticks(y_pos22,objects22)
plt.title("Query 22 performance")
plt.ylabel("Seconds")
plt.show()


cur.close()
conn.commit()
        






			







