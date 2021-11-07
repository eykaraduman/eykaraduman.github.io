---
title: Polyline Parçalarını Nasıl Sileriz?
permalink: /acad/delete-polyline-vertex/
published: true
classes: wide
categories:
  - ObjectARX
---

 <strong>AcDb3dPolyline</strong>  ve <strong>AcDb2dPolyline</strong> nesneleri karmaşık (=kompleks) nesnelerdir. Karmaşıklıktan kasıt bu iki nesnenin parçalarını oluşturan <strong>AcDb3dPolylineVertex</strong> ile <strong>AcDb2dVertex</strong> alt nesnelerinin veritabanında ayrıca varolmalarıdır. <strong>AcDbPline</strong> ise tersine parçalarıyla birlikte tek bir nesnedir. 

 AcDb3dPolyline, AcDb2dPolyline ve AcDbPolyline nesnelerini ait parçaları silmek için <strong>DeleteVertex()</strong> metodunda aşağıdaki yol izlenmiştir:

<ul><li><strong>acedNEntSelP(…)</strong> metodu kullanılarak AcDb3dPolylineVertex, AcDb2dVertex ve AcDbPolyline nesnelerine erişilmiş</li><li>Seçilen nesne AcDb3dPolylineVertex ya da AcDb2dVertex ise AcDbObject <strong>erase()</strong> metodu kullanılarak, AcDbPolyline içinse seçilen noktanın hangi parça üzerinde olduğu AcDbPolyline <strong>getClosestPointTo(…)</strong> ve <strong>onSegAt(..)</strong> metotlarıyla araştırılarak bulunan parça <strong>removeVertexAt(…)</strong> metoduyla silinmiştir.</li></ul>

```cpp
 void DeleteVertex(void)  
 {  
      ads_name ent;  
      ads_point ptres;  
      int pickflag;  
      ads_matrix xformres;  
      struct resbuf *refstkres;  
      pickflag=0;  
      if (acedNEntSelP(_T("\nSelect a nested entity: "), ent, ptres, pickflag, xformres,  refstkres) == RTNORM)   
      {  
           acutRelRb(refstkres);  
           AcDbObjectId objId;  
           AcDbObject *obj;  
           if(acdbGetObjectId(objId, ent) != Acad::eOk)  
           {  
                acutPrintf(_T("\nError getting entity."));   
                return;                 
           }  
           if(acdbOpenAcDbObject( obj, objId, AcDb::kForRead) != Acad::eOk)  
           {  
                acutPrintf(_T("\nError opening entity."));   
                return;                 
           }  
           if (obj->isKindOf(AcDb3dPolylineVertex::desc()) || obj->isKindOf(AcDb2dVertex::desc()))   
           {  
                if(obj->upgradeOpen() != Acad::eOk)  
                {  
                     acutPrintf(_T("\nError opening polyline vertex."));   
                     obj->close();  
                     return;  
                }  
                obj->erase();  
                obj->close();                 
           }  
           else if(obj->isKindOf(AcDbPolyline::desc()))  
           {  
                ads_point ptWcs;  
                AcGePoint3d searchPnt;  
                acdbUcs2Wcs(ptres, ptWcs, Adesk::kFalse);  
                AcDbPolyline *pPLine = static_cast<AcDbPolyline*>(obj);  
                pPLine->upgradeOpen();  
                pPLine->getClosestPointTo(asPnt3d(ptWcs), searchPnt);  
                AcGePoint2d searchPnt2d(searchPnt.x, searchPnt.y);   
                for(unsigned int v=0; pPLine->numVerts() > v ; v++)  
                {  
                  double segParam;  
                  if(pPLine->onSegAt(v, searchPnt2d, segParam))   
                  {  
                       pPLine->removeVertexAt(v);  
                       break;  
                  }  
                }  
                pPLine->close();  
           }  
           else  
           {  
                acutPrintf(_T("\nYou did not select a 2d, 3d or Lw polyline."));   
                obj->close();  
                return;  
           }             
      }  
 }  
```

