---
title: Polyline Köşe Nokta Koordinatlarını Dosyaya Kaydetme
permalink: /acad/polyline-vertex-to-file/
published: true
classes: wide
categories:
  - ObjectARX
---

`AcDbPolyline`, `AcDb2dPolyline` ve `AcDb3dPolyline` nesnelerine ait köşe nokta koordinatlarının *.csv, *.txt uzantılı dosyalara nasıl yazdırılabileceğini `WriteCoordinatesOfPolylineToFile()` metodunda bulabilirsiniz.

```c++
void WriteCoordinatesOfPolylineToFile()  
{  
      AcDbObjectId objId = AcDbObjectId::kNull;   
      AcDbEntity *pObj = NULL;  
      Acad::ErrorStatus es = Acad::eOk;  
      ads_name ename;  
      ads_point pickpt;       
      if(acedEntSel(_T("\nPolyline seçin: "), ename, pickpt) == RTNORM)  
      {  
           acdbGetObjectId(objId, ename);   
           if ((es = acdbOpenAcDbEntity(pObj, objId, AcDb::kForRead)) != Acad::eOk)   
           {  
                acutPrintf(_T("\nNesne seçilmedi."));   
                return;  
           }       
           AcGePoint3dArray vertices;       
           vertices.setLogicalLength(0);  
           if(pObj->isKindOf(AcDbPolyline::desc()))  
           {  
                AcDbPolyline* poly = static_cast&lt;AcDbPolyline*>(pObj);  
                // LwPolyline nesnesi köşe noktalarının elde edilmesi  
                vertices = getLwPolyVertices(poly);  
           }  
           else if(pObj->isKindOf(AcDb3dPolyline::desc()))  
           {  
                AcDb3dPolyline* poly3D = static_cast&lt;AcDb3dPolyline*>(pObj);  
                // 3d Polyline nesnesi köşe noktalarının elde edilmesi  
                vertices = get3dPolyVertices(poly3D);  
           }  
           else if(pObj->isKindOf(AcDb2dPolyline::desc()))  
           {  
                AcDb2dPolyline* poly2D = static_cast&lt;AcDb2dPolyline*>(pObj);  
                // 2d Polyline nesnesi köşe noktalarının elde edilmesi  
                vertices = get2dPolyVertices(poly2D);  
           }  
           else  
           {  
                acutPrintf(_T("\nSeçtiğiniz nesne polyline değil!"));   
                pObj->close();  
                return;  
           }  
           pObj->close();  
           const ACHAR* filea = _T("PoylineKoseler.csv");  
           const ACHAR * dlgname = _T("Bir dosya seçin");  
           struct resbuf* result = NULL;  
           acedGetFileNavDialog(filea, NULL, _T("txt;csv"), dlgname, 33, &amp;result);  
           ofstream file;  
           file.open(result->resval.rstring);  
           acutRelRb(result);  
           for(int i = 0; i &lt; vertices.length(); i++)  
           {                 
                file << i+1 << "," << fixed << vertices[i].x << "," << vertices[i].y << "," << vertices[i].z << endl;  
           }  
           file.close();  
      }  
}  
```

Polyline köşe nokta koordinatlarını ise aşağıdaki metotlarla elde etmek mümkün.

```c++
 AcGePoint3dArray get3dPolyVertices(AcDb3dPolyline *poly3D)  
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
 AcGePoint3dArray get2dPolyVertices(AcDb2dPolyline *poly2D)  
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
 AcGePoint3dArray getLwPolyVertices(AcDbPolyline *poly)  
 {  
      AcGePoint3dArray tmpArray;       
      tmpArray.setLogicalLength(0);  
      int nOfVerts = poly->numVerts();  
      for (int vertexNumber = 0; vertexNumber &lt; nOfVerts; vertexNumber++)   
      {  
           AcGePoint2d pnt;  
           poly->getPointAt(vertexNumber, pnt);  
           tmpArray.append(AcGePoint3d(pnt.x, pnt.y, 0.0));  
      }  
      return tmpArray;  
 } 
```

