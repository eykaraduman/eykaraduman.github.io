---
title: 2B Polyline Nesnelerini LwPolyline Nesnelerine Çevirme
permalink: /acad/2dpoly-to-lwpoly/
published: true
classes: wide
categories:
  - ObjectARX
---

AutoCAD PLINE komutu, PLINETYPE = 0 değeri için Polyline (= AcDb2dPolyline), PLINETYPE = 1 ve 2 değerleri içinse Lwpolyline nesnesi çizebilmemizi sağlar. <strong>ConvertPoly2dToLwPolyline(…)</strong> fonksiyonunu,  AcDb2dPolyline nesnelerini AcDbPolyline nesnelerine çevirmekte kullanabilirsiniz. <strong>makePolyline(…)</strong>, <strong>addToCurrentSpace(…)</strong> ve <strong>openCurrentSpaceBlock(…)</strong> fonksiyonları için <a href="http://www.eykaraduman.github.io/acad/3dpoly-to-lwpoly/">"3B Polyline Nesnelerini LwPolyline Nesnelerine Çevirme”</a> adlı yazıya bakın.

```cpp
 void PolyConverter::ConvertPoly2dToLwPolyline(void)  
 {  
      // 2b polyline nesnelerinin seçimi için filtre oluşturulması  
      struct resbuf eb;  
      TCHAR sbuf[10];  
      eb.restype = 0;  
      wcscpy(sbuf, _T("POLYLINE"));  
      eb.resval.rstring = sbuf;  
      eb.rbnext = NULL;  
      ads_name ss;  
      if (acedSSGet(_T("X"), NULL, NULL, eb, ss) != RTNORM)  
      {  
           acutRelRb(eb);  
           return;  
      }  
      // resbuf'ın serbest bırakılması  
      acutRelRb(eb);  
      // Seçilen nesnelerin sayısının bulunması  
      long length = 0;  
      if ((acedSSLength(ss, length) != RTNORM) || (length == 0))   
      {  
           acedSSFree(ss);  
           return;  
      }  
      ads_name ent;  
      AcDbObjectId id = AcDbObjectId::kNull;  
      AcDbDatabase *pCurDb = acdbCurDwg();  
      AcApDocument *pDoc = acDocManager->mdiActiveDocument();  
      // Aktif dökümanın kilitlenmesi  
      acDocManager->lockDocument(pDoc);  
      acedSetStatusBarProgressMeter(_T("Nesneler çevriliyor.Lüffen işlemin bitmesini bekleyiniz..."), 0, length);  
      // İşlem yığını yöneticisinin başlatılması  
      AcTransaction *tr = acTransactionManagerPtr()->startTransaction();  
      int abort = 0;  
      for (long i = 0; i < length; i++)   
      {  
           if (acedSSName(ss, i, ent) != RTNORM) continue;  
           if (acdbGetObjectId(id, ent) != Acad::eOk) continue;  
           AcDbObject *pEnt = NULL;  
           // Seçilen nesnenin yazma amaçlı açılması  
           if(tr->getObject((AcDbObject *)pEnt, id, AcDb::kForWrite) != Acad::eOk)  
                continue;  
           //2dPolyline nesnesinin elde edilmesi  
           if(pEnt->isKindOf(AcDb2dPolyline::desc()))  
           {  
                AcDb2dPolyline* poly2D = static_cast<AcDb2dPolyline*>(pEnt);  
                // 2dPolyline nesnesi köşe noktalarının elde edilmesi  
                AcGePoint3dArray vertices = get2dPolyVertices(poly2D);  
                AcDbPolyline *pline;  
                AcDbObjectId objId = AcDbObjectId::kNull;  
                // Lwpolyline nesnesinin oluşturulması  
                if(makePolyline(static_cast<AcDbEntity*>(pEnt)->layer(), vertices, objId) == Acad::eOk)  
                {                      
                     pEnt->erase();            
                }  
           }  
           acedSetStatusBarProgressMeterPos(i);  
           // Eğer kullanıcı işlemi iptal ederse işlem yığınını iptal edilmesi  
           abort = acedUsrBrk();  
           if(abort == 1)  
           {                  
                acutPrintf(_T("\nİşlem iptal edildi..."));  
                break;  
           }  
      }  
      if(abort == 0)   
           acTransactionManagerPtr()->endTransaction();  
      else  
           acTransactionManagerPtr()->abortTransaction();  
      acedRestoreStatusBar();  
      acDocManager->unlockDocument(pDoc);  
      // Seçim setinin serbest bırakılması  
      acedSSFree(ss);  
 } 
```

2dPolyline nesnesi köşe nokta listesinin elde edilmesi:

```cpp
 AcGePoint3dArray PolyConverter::get2dPolyVertices(AcDb2dPolyline *poly2D)  
 {  
      AcGePoint3dArray tmpArray;       
      tmpArray.setLogicalLength(0);  
      AcDbObjectIterator *pVertIter= poly2D->vertexIterator();  
      AcDb2dVertex *pVertex;   
      AcGePoint3d location;   
      AcDbObjectId vertexObjId;   
      for (int vertexNumber = 0; !pVertIter->done(); vertexNumber++, pVertIter->step())   
      {   
           vertexObjId = pVertIter->objectId();   
           acdbOpenObject(pVertex, vertexObjId, AcDb::kForRead);   
           location = pVertex->position();   
           tmpArray.append(location);  
           pVertex->close();   
      }   
      delete pVertIter;  
      return tmpArray;  
 } 
```

