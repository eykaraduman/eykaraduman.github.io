---
title: Spline Nesnelerini Lwpolyline Nesnelerine Çevirme
permalink: /acad/spline-to-lwpoly/
published: true
classes: wide
categories:
  - ObjectARX
---

ObjectArx <strong>AcDbSpline</strong> nesnesi, <strong>toPolyline(…)</strong> adlı bir methodu zaten içeriyor. <strong>ConvertSplineToLwPolyline(…)</strong> fonksiyonu ise bu methodu kullanarak, çizimdeki Spline nesnelerini Lwpolyline nesnelerine çevirmekten başka bir şey yapmıyor.

```cpp
 void PolyConverter::ConvertSplineToLwPolyline(void)  
 {  
      // Spline nesnelerinin seçimi için filtre oluşturulması  
      struct resbuf eb;  
      TCHAR sbuf[10];  
      eb.restype = 0;  
      wcscpy(sbuf, _T("SPLINE"));  
      eb.resval.rstring = sbuf;  
      eb.rbnext = NULL;  
      ads_name ss;  
      if (acedSSGet(NULL, NULL, NULL, eb, ss) != RTNORM)  
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
           //Spline nesnesinin elde edilmesi  
           if(pEnt->isKindOf(AcDbSpline::desc()))  
           {  
                AcDbSpline *pSpline = static_cast<AcDbSpline*>(pEnt);  
                AcDbPolyline *pPline;  
                AcDbObjectId objId = AcDbObjectId::kNull;  
                Acad::ErrorStatus es = pSpline->toPolyline((AcDbCurve *)pPline);  
                if(es == Acad::eOk)  
                {  
                     if(addPolyline(pCurDb, tr, static_cast<AcDbEntity*>(pEnt)->layer(), pPline, objId) == Acad::eOk)  
                     {                      
                          pEnt->erase();            
                     }  
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

```cpp
 Acad::ErrorStatus PolyConverter::addPolyline(AcDbDatabase *db, AcTransaction *tr, ACHAR *LayerName, AcDbPolyline *pPline, AcDbObjectId Id)  
 {  
   Acad::ErrorStatus es;  
      pPline->setLayer(LayerName);  
      es = addToCurrentSpace(pPline, acdbCurDwg(), tr, Id);  
       return es;  
 }
```

