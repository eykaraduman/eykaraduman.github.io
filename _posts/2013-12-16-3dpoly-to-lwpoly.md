---
title: 3B Polyline Nesnelerini LwPolyline Nesnelerine Çevirme
permalink: /acad/3dpoly-to-lwpoly/
published: true
classes: wide
categories:
  - ObjectARX
---

Aşağıda kaynak kodunu ve derlenmiş indirme bağlantısını bulabileceğiniz ObjectARX uygulamasını, 3D Polyline (=POLYLINE) nesnelerini Polyline (=LWPOLYLINE) nesnelerine çevirmekte kullanabilirsiniz. <strong>ConvertPoly3dToLwPolyline</strong> fonksiyonunu ObjectARX ile işlem yığını yöneticisinin (AcTransactionManager) kullanımının bir örneği olarak da inceleyebilirsiniz.

```cpp
void PolyConverter::ConvertPoly3dToLwPolyline(void)  
 {  
      // 3b polyline nesnelerinin seçimi için filtre oluşturulması  
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
           //3dPolyline nesnesinin elde edilmesi  
           if(pEnt->isKindOf(AcDb3dPolyline::desc()))  
           {  
                AcDb3dPolyline* poly3D = static_cast<AcDb3dPolyline*>(pEnt);  
                // 3dPolyline nesnesi köşe noktalarının elde edilmesi  
                AcGePoint3dArray vertices = get3dPolyVertices(poly3D);  
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

3dPolyline nesnesi köşe nokta listesinin elde edilmesi: 

```cpp
 AcGePoint3dArray PolyConverter::get3dPolyVertices(AcDb3dPolyline *poly3D)  
 {  
      AcGePoint3dArray tmpArray;       
      tmpArray.setLogicalLength(0);  
      AcDbObjectIterator *pVertIter= poly3D->vertexIterator();  
      AcDb3dPolylineVertex *pVertex;   
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

Polyline nesnesinin oluşturulması:

```cpp
Acad::ErrorStatus PolyConverter::makePolyline(AcDbDatabase *db, AcTransaction *tr, ACHAR* LayerName, AcGePoint3dArray vertices, AcDbObjectId Id)  
 {  
   int ptCount;  
      ptCount = vertices.length();  
   AcDbPolyline* pline = new AcDbPolyline(ptCount);  
   Acad::ErrorStatus es;  
   AcGePoint2d pt;  
   for (int i=0;i<ptCount;i++)  
      {  
     pt.set(vertices[i].x, vertices[i].y);  
     pline->addVertexAt(i,pt);   
   }  
      pline->setElevation(vertices[0].z);  
      pline->setLayer(LayerName);  
      es = addToCurrentSpace(pline, acdbCurDwg(), tr, Id);  
       return es;  
 }
```

Oluşturulan polyline nesnesinin geçerli uzaya eklenmesi:

```cpp
 Acad::ErrorStatus PolyConverter::addToCurrentSpace(AcDbEntity *newEnt, AcDbDatabase *db, AcTransaction *tr, AcDbObjectId Id)  
 {  
   ASSERT(newEnt != NULL);  
      ASSERT(db != NULL);  
   AcDbBlockTableRecord *blkRec = openCurrentSpaceBlock(AcDb::kForWrite, db, tr);  
   ASSERT(blkRec != NULL);  
   if (blkRec == NULL)  
     return Acad::eInvalidInput;  
   Acad::ErrorStatus es = blkRec->appendAcDbEntity(Id,newEnt);       
   if (es == Acad::eOk)   
      {  
     es = acTransactionManagerPtr()->addNewlyCreatedDBRObject(newEnt, true);       
   }  
   return es;  
 } 
```

```cpp
AcDbBlockTableRecord *PolyConverter::openCurrentSpaceBlock(AcDb::OpenMode mode, AcDbDatabase* db, AcTransaction* tr)  
 {  
      ASSERT(db != NULL);  
      ASSERT(tr != NULL);  
   AcDbBlockTableRecord* blkRec;  
      Acad::ErrorStatus es = tr->getObject((AcDbObject *)blkRec, db->currentSpaceId(), AcDb::kForWrite);  
   if (es != Acad::eOk)  
     return NULL;  
   else  
     return blkRec;  
 } 
```

