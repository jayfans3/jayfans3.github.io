---
layout: post
title: hbaseScan
categories:
- 人
tags:
- 书
---



#hbaseScan

sequence:


client
-----

->HTable.getScanner(Scan scan)

->ClientScanner.nextScanner(int nbRows, boolean done)

->ClientScanner.getScannerCallable(byte[] localStartKey, int nbRows)

->ScannerCallable.call()//rpc远程regionserver上调用

->this.server.openScanner(this.location.getRegionInfo().getRegionName(),this.scan)

server:
-------------------------------------

->打开scanner:HRegionServer.openScanner

->通知协处理器：r.getCoprocessorHost().preScannerOpen(scan)

->scanner生成：HRegion.getScanner(Scan scan,List<KeyValueScanner> additionalScanners)

->定位startrow：RegionScannerImpl: HRegion.instantiateRegionScanner(Scan scan,List<KeyValueScanner> additionalScanners)

:->store.getScanner(scan,entry.getValue());

  - ScanQueryMatcher.match->MatchCode处理当前的kv

  - hfile的迭代操作：HFile.Reader
  - index定位keyvalue:ScannerV2(HFileScanner)
  - 返回List<KeyValueScanner> = StoreFileScanner+memStoreScanners
  - selectScannersFrom（）：passesTimerangeFilter，passesBloomFilter
  - scanner.requestSeek(matcher.getStartKey(), false, true);
  - **seekAtOrAfter->HFileReaderV2.AbstractScannerV2.seekTo->ScannerV2.blockBuffer->startRow**
  - hfilescanner.cur = hfs.getKeyValue() = new KeyValue(blockBuffer.array(),blockBuffer.arrayOffset() + blockBuffer.position());
  - 
  - heap = new KeyValueHeap(scanners, store.comparator)
  - this.current = pollRealKV();
  - HRegionServer.addScanner(RegionScannerImpl)->map中


HRegionServer.next->RegionScannerImpl.next->KeyValueHeap.next->StoreScanner.next
------------------

StoreScanner.next(List<KeyValue>, int):

SEEK_NEXT_COL
------------------------------
INCLUDE_AND_SEEK_NEXT_ROW
------------------------------
SEEK_NEXT_COL
------------------------------
INCLUDE_AND_SEEK_NEXT_ROW 