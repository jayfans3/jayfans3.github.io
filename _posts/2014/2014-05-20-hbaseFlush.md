---
layout: post
title: Hbase Flush
categories:
- ��
tags:
- ��
---



#Hbase Flush

���


stripe compaction

ì�ܣ�

0. ��������->region��->store��->memstore��->flush��->compact��->Ӱ���д
1. ĳrowkey��Χ����put����Ҫ����store��major compact.
2. split���۴�

�����

level:

2. ��level0��level1:��ʱmem flush,bulkload�ļ���stripe�ļ�
7. level0-1�����ϲ���
4. �ݴ���level1����level0����compaction
9. �㷨leve0-n:1.ͬһ��rowkey����2.�ߺϲ���ɾ����lru��leve�ͣ��ֲ�ϲ��������еĺϲ���
10. compaction�籩����Ҫ��

level1-stripe:
1. region�°�rowkey����stripe
3. ���ֹ���:��С�Զ��У�miniregions����
5. getɨ����ۼ���level1��
6. scan stripe����//snapshot�����Ѿ���������
8. ����split,ֱ�Ӷ�λ����




---------------


.Yali
2.jiekout jieou 
3.chengdu drquery git branch �Cr 
4. ������������ 
5.��ҳ 




-------
0.flush ˢregion���ģ�
1. ����region��memstore 
2. bestflushableregionѡregion:store��hbase.hstore.blockingStoreFiles������memstore����region��. ���������
3. bestanyregion maxMemstore
4. 3��2�����Ļ�����compactionҲѡ���reigon,3������ѡ2
5. regionһ�� mvcc����д��������Ҫдmemstore
6. flush����row��������snapshot->memstore.
7. ��mvccд����memstore�� mvcc���汾����ǰд�汾
8. mvcc��memstore��updatelock�����ͷţ�flushҪ�ȴ�mvccд���ͷţ��ȴ�Ŀǰ��д�汾��
