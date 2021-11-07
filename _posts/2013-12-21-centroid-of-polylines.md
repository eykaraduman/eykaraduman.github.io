---
title: Kapalı Çizgilerin Ağırlık Merkezini İşaretleme
permalink: /acad/cntroid-of-polylines/
published: true
classes: wide
categories:
  - ObjectARX
tags:
  - MFC
---

AutoCAD ortamında kapalı bir nesnenin ağırlık merkezini bulmak istediğimizde, önce nesneyi bir bölgeye (region) çevirir, daha sonra da <strong>MASSPROP (Tools->Inquiry->Region/Mass Properties)</strong> komutuyla bu bölgenin özelliklerini komut satırında listeleriz.  Aşağıda kaynak kodunu verdiğim <strong>FindCentroid(…)</strong> fonksiyonu, AcDbPolyline, AcDb2dPolyline, AcDb3dPolyline (kapalı olmaları koşuluyla) ve AcDbCircle nesnelerini, MASSPROP komutunu kullanmanıza gerek kalmaksızın bir bölgeye çevirip ağırlık merkezine bir nokta yerleştiriyor.

<img src="{{ "/assets/images/MarkCentroid.png" | absolute_url }}"  alt="Şekil-1" style="width:25%">

<figure>
  <figcaption>Şekil-1</figcaption>
</figure>
```cpp
void Centroid::FindCentroid(bool drawRegion, ACHAR *regionLayer, ACHAR *pointLayer)  
 {  
      ads_name objName;  
      ads_point pickpt;  
      Acad::ErrorStatus es;  
      AcDbEntity* pEntity = NULL;       
      AcDbObjectId objId = AcDbObjectId::kNull;  
      // AcDbRegion getAreaProp(...) metodu için gerekli değişkenler  
      AcDbVoidPtrArray curveSegs, regions;  
      double perimeter, area;  
      AcGePoint3d origin;  
      AcGeVector3d xAxis, yAxis;  
      AcGePoint2d centroid;  
      double momInertia[2], prodInertia, prinMoments[2], radiiGyration[2];  
      AcGeVector2d prinAxes[2];  
      AcGePoint2d extentsLow, extentsHigh;  
      AcGePlane plane;  
      // Ağırlık merkezi araştırılacak nesnenin seçilmesi  
      if(acedEntSel(_T("\nSelect object: "), objName, pickpt) != RTNORM)   
      {  
           acutPrintf(_T("\nNothing selected."));  
           return;  
      } else   
      {  
           // Nesne isminin elde edilmesi  
           if(acdbGetObjectId(objId, objName) != Acad::eOk)   
                return;  
           // Nesnenin okuma amaçlı açılması  
           es = acdbOpenAcDbEntity(pEntity, objId, AcDb::kForRead);  
           if (es == Acad::eOk)   
           {  
                // Nesnenin istenilen tipte olup olmadığının araştırılması  
                if (pEntity->isKindOf(AcDbPolyline::desc())   
                     || pEntity->isKindOf(AcDb3dPolyline::desc())  
                     || pEntity->isKindOf(AcDb2dPolyline::desc())   
                     || pEntity->isKindOf(AcDbCircle::desc())  
                     )   
                {  
                     curveSegs.append(static_cast<void*>(pEntity));  
                }   
                else   
                {  
                     pEntity->close();  
                     acutPrintf(_T("\nError: Invalid object."));  
                     return;  
                }  
           }   
           else   
           {  
                acutPrintf(_T("\nError: Can not obtain object."));  
                return;  
           }  
           es = AcDbRegion::createFromCurves(curveSegs, regions);  
           if (es != Acad::eOk)   
           {  
                pEntity->close();  
                acutPrintf(_T("\nError: Select a closed line."));  
                return;  
           }   
           else   
           {  
                // AcDbRegion'un oluşturulması  
                AcDbRegion *pRegion;  
                pRegion = static_cast<AcDbRegion*>(regions[0]);  
                pRegion->getPlane(plane);  
                plane.getCoordSystem(origin, xAxis, yAxis);  
                origin.x = 0.0;  
                origin.y = 0.0;  
                // AcDbRegion alan özelliklerinin elde edilmesi  
                es = pRegion->getAreaProp(origin, xAxis, yAxis, perimeter, area, centroid, momInertia,  
                     prodInertia, prinMoments, prinAxes, radiiGyration, extentsLow, extentsHigh);  
                // Blok tablosuna eklenecek nesneler için AcDbBlockTableRecord işaretçisine erişim  
                AcDbBlockTable *pBlockTable;  
                acdbHostApplicationServices()->workingDatabase()->getSymbolTable(pBlockTable, AcDb::kForRead);  
                AcDbBlockTableRecord *pBlockTableRecord;  
                pBlockTable->getAt(ACDB_MODEL_SPACE, pBlockTableRecord, AcDb::kForWrite);  
                pBlockTable->close();  
                // AcDbRegion nesnesinin blok tablosuna eklenmesi  
                if(drawRegion)  
                {  
                     AcDbObjectId regionId;  
                     pRegion->setLayer(regionLayer);  
                     es = pBlockTableRecord->appendAcDbEntity(regionId, pRegion);       
                }  
                // Ağırlık merkezini işaretlemek için AcDbPoint   
                // oluşturulması ve blok tablosuna eklenmesi  
                AcDbPoint *pPnt = new AcDbPoint;  
                pPnt->setDatabaseDefaults();  
                pPnt->setPosition(AcGePoint3d::AcGePoint3d(centroid.x,centroid.y,0.0));  
                pPnt->setLayer(pointLayer);  
                AcDbObjectId pointId;  
                pBlockTableRecord->appendAcDbEntity(pointId, pPnt);       
                // Blok tablo kaydının kapatılması  
                pBlockTableRecord->close();  
                // Nesnelerin kapatılması  
                pPnt->close();  
                pRegion->close();  
                pEntity->close();  
                if(es == Acad::eOk)  
                {  
                     acutPrintf(_T("\nCentroid (X, Y, Z) = (%.4f, %.4f, %.4f)"), centroid.x, centroid.y, origin.z);  
                     acutPrintf(_T("\nArea = %.4f | Perimeter = %.4f"), area, perimeter);  
                }  
                // Görünümün güncellemesi  
                actrTransactionManager->flushGraphics();  
                acedUpdateDisplay();  
           }       
      }  
      return;  
 }  
```

<strong>FindCentroid(…)</strong> fonksiyonunun kullanımı için hazırladığım diyalog kutusuna ait <strong>MarkCentroidDialog.cpp</strong> dosyası ise aşağıdaki gibi:

```c++
//-----------------------------------------------------------------------------  
 //----- MarkCentroidDialog.cpp : Implementation of MarkCentroidDialog  
 //-----------------------------------------------------------------------------  
 #include "StdAfx.h"  
 #include "resource.h"  
 #include "MarkCentroidDialog.h"  
 #include "Centroid.h"  
 //-----------------------------------------------------------------------------  
 IMPLEMENT_DYNAMIC (MarkCentroidDialog, CAcUiDialog)  
 BEGIN_MESSAGE_MAP(MarkCentroidDialog, CAcUiDialog)  
      ON_MESSAGE(WM_ACAD_KEEPFOCUS, OnAcadKeepFocus)  
      ON_BN_CLICKED(IDC_BUTTON_SELECT_OBJECT, MarkCentroidDialog::OnClickedButtonSelectObject)  
 ON_BN_CLICKED(IDC_CHECK_DRAW_REGION, MarkCentroidDialog::OnClickedCheckDrawRegion)  
 END_MESSAGE_MAP()  
 //-----------------------------------------------------------------------------  
 MarkCentroidDialog::MarkCentroidDialog (CWnd *pParent /*=NULL*/, HINSTANCE hInstance /*=NULL*/) : CAcUiDialog (MarkCentroidDialog::IDD, pParent, hInstance) {  
 }  
 //-----------------------------------------------------------------------------  
 void MarkCentroidDialog::DoDataExchange (CDataExchange *pDX) {  
      CAcUiDialog::DoDataExchange (pDX) ;  
      DDX_Control(pDX, IDC_BUTTON_SELECT_OBJECT, m_btnSelect);  
      DDX_Control(pDX, IDC_CHECK_DRAW_REGION, m_drawRegion);  
      DDX_Control(pDX, IDC_COMBO_POINT_LAYER, m_pntLayerList);  
      DDX_Control(pDX, IDC_COMBO_REGION_LAYER, m_regionLayerList);  
 }  
 //-----------------------------------------------------------------------------  
 //----- Needed for modeless dialogs to keep focus.  
 //----- Return FALSE to not keep the focus, return TRUE to keep the focus  
 LRESULT MarkCentroidDialog::OnAcadKeepFocus (WPARAM, LPARAM) {  
      return (TRUE) ;  
 }  
 BOOL MarkCentroidDialog::OnInitDialog()  
 {  
   SetDialogName(_T("PolylineProperties:MarkCentroidDialog"));  
      CAcUiDialog::OnInitDialog();  
      m_btnSelect.AutoLoad();  
      GetLayerNames();  
      GetData();  
      return TRUE; // return TRUE unless you set the focus to a control  
 }  
 void MarkCentroidDialog::GetLayerNames()   
 {  
   AcDbLayerTable *pLayerTable;  
   acdbHostApplicationServices()->workingDatabase()->getSymbolTable(pLayerTable, AcDb::kForRead);  
   const ACHAR *pName;  
   AcDbLayerTableIterator *pItr;  
   if (pLayerTable->newIterator(pItr) == Acad::eOk)   
      {  
     while (!pItr->done())   
           {  
       AcDbLayerTableRecord *pRecord;  
       if (pItr->getRecord(pRecord, AcDb::kForRead) == Acad::eOk)   
                {  
         pRecord->getName(pName);  
         m_pntLayerList.InsertString(-1, pName);  
                     m_regionLayerList.InsertString(-1, pName);  
         pRecord->close();  
       }  
       pItr->step();  
     }  
   }  
   pLayerTable->close();  
 }  
 void MarkCentroidDialog::OnClickedButtonSelectObject()  
 {  
      int nIndexPoint = m_pntLayerList.GetCurSel();  
      int nIndexRegion = m_regionLayerList.GetCurSel();  
      CString regionLayer = _T("");  
      CString pointLayer= _T("");  
      m_regionLayerList.GetLBText(nIndexRegion, regionLayer);  
      m_pntLayerList.GetLBText(nIndexPoint, pointLayer);  
      BeginEditorCommand();  
      Centroid cen;  
      cen.FindCentroid(m_drawRegion.GetCheck() ? true:false,   
           regionLayer.GetBuffer(regionLayer.GetLength()),   
           pointLayer.GetBuffer(pointLayer.GetLength()));       
      CompleteEditorCommand();  
 }  
 void MarkCentroidDialog::OnOK()  
 {  
      CAcUiDialog::OnOK();  
      SetData();  
 }  
 void MarkCentroidDialog::SetData()  
 {  
      int nIndexPoint = m_pntLayerList.GetCurSel();  
      int nIndexRegion = m_regionLayerList.GetCurSel();  
      CString regionLayer = _T("");  
      CString pointLayer= _T("");  
      m_regionLayerList.GetLBText(nIndexRegion, regionLayer);  
      m_pntLayerList.GetLBText(nIndexPoint, pointLayer);  
      SetDialogData(_T("REGIONLAYER"), regionLayer.GetBuffer(regionLayer.GetLength()));  
      SetDialogData(_T("POINTLAYER"), pointLayer.GetBuffer(pointLayer.GetLength()));  
      SetDialogData(_T("DRAWREGION"), m_drawRegion.GetCheck() ? 1:0);  
 }  
 void MarkCentroidDialog::GetData(void)  
 {  
      CString regionLayer;  
      if (GetDialogData(_T("REGIONLAYER"), regionLayer))  
      {  
           int nIndexRegion = m_regionLayerList.FindString(-1, regionLayer);  
           if (nIndexRegion == CB_ERR)  
           {  
                m_regionLayerList.SetCurSel(0);  
           }  
           else  
           {  
                m_regionLayerList.SetCurSel(nIndexRegion);  
           }  
      }  
      CString pointLayer;  
      if (GetDialogData(_T("POINTLAYER"), pointLayer))  
      {  
           int nIndexPoint = m_pntLayerList.FindString(-1, pointLayer);  
           if (nIndexPoint == CB_ERR)  
           {  
                m_pntLayerList.SetCurSel(0);  
           }  
           else  
           {  
                m_pntLayerList.SetCurSel(nIndexPoint);  
           }  
      }  
      DWORD drawRegion;  
      if (GetDialogData(_T("DRAWREGION"), drawRegion))  
      {  
           m_drawRegion.SetCheck(drawRegion);  
           if(drawRegion == 1)  
                m_regionLayerList.EnableWindow(true);  
           else  
                m_regionLayerList.EnableWindow(false);  
      }  
      else  
      {  
           m_drawRegion.SetCheck(0);  
           m_regionLayerList.EnableWindow(false);  
      }  
 }  
 void MarkCentroidDialog::OnClickedCheckDrawRegion()  
 {  
      if((m_drawRegion.GetCheck() ? true:false) == true)  
           m_regionLayerList.EnableWindow(true);  
      else  
           m_regionLayerList.EnableWindow(false);  
 }  
```

