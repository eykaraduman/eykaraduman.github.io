---
title: Text, MText Nesneleriyle Aritmetik İşlemler 
permalink: /acad/arithmetic-for-text-objects/
published: true
classes: wide
categories:
  - ObjectARX
---

Calculate() metodu, AcDbText ve AcDbMText nesneleri üzerinde toplama ve çarpma işlemleri yapmak için tasarlanmıştır.
Calculate(), SelectOperationType(…), SelectResultType(…), WriteResult(…) metodlarını incelerseniz,

- Seçim setleri, seçim seti filtrelerinin (acedSSGet, acedSSLength, acedSSName)
- AcDb global dönüştürme fonkiyonlarının (acdbDisToF,acdbRToS)
- Kullanıcıdan bilgi almaya yarayan anahtar sözcüklerlerin (acedInitGet, acedGetKword) kullanımına dair örnekler bulabilirsiniz.

```cpp
void TextMath::Calculate()  
 {  
      int addition = 0;  
      if(SelectOperationType(addition) != RTNORM)  
           return;  
      resbuf* pTextFilter = NULL;  
      pTextFilter = acutBuildList(RTDXF0, ACRX_T("TEXT,MTEXT"), NULL);  
      ads_name sset;  
      long ssetLen;  
      acutPrintf(_T("Select Text, MText objects"));  
      if(acedSSGet(NULL, NULL, NULL, pTextFilter, sset) != RTNORM){  
           acutPrintf(_T("\nNothing selected."));  
           acutRelRb(pTextFilter);  
           return;  
      }  
      if(acedSSLength(sset, &ssetLen) != RTNORM){  
           acutPrintf(_T("\nNothing selected."));  
           acutRelRb(pTextFilter);  
           acedSSFree(sset);  
           return;  
      }  
      Acad::ErrorStatus es;  
      AcDbEntity *ent;  
      long rejectedCount = 0;  
      ads_real x = NULL;  
      ads_real y = NULL;  
      for (int i = 0; i < ssetLen; i++) {  
           ads_name entName;  
           acedSSName(sset, i , entName);  
           AcDbObjectId objId;  
           acdbGetObjectId(objId, entName);  
           es = acdbOpenAcDbEntity(ent, objId, AcDb::kForRead);  
           if (es == Acad::eOk) {                 
                if (ent->isKindOf(AcDbText::desc())) {  
                     AcDbText *pText = static_cast<AcDbText*>(ent);  
                     if(acdbDisToF(pText->textString(), 2, &x) == RTNORM){  
                          if(x==0.0){  
                               pText->close();  
                               continue;  
                          }  
                          addition == 1 ? y+=x : (y==NULL ? y=x : y*=x);  
                     }  
                     else{  
                          rejectedCount++;  
                     }  
                     pText->close();  
                }  
                else if (ent->isKindOf(AcDbMText::desc())){  
                     AcDbMText *pMText = static_cast<AcDbMText*>(ent);  
                     if(acdbDisToF(pMText->contents(), 2, &x) == RTNORM){  
                          if(x==0.0){  
                               pMText->close();  
                               continue;  
                          }  
                          addition == 1 ? y+=x : (y==NULL ? y=x : y*=x);  
                     }  
                     else{  
                          rejectedCount++;  
                     }  
                     pMText->close();  
                }  
                ent->close();  
           }  
      }  
      acutRelRb(pTextFilter);  
      acedSSFree(sset);  
      if(y != NULL){  
           int resType = 0;  
           if(SelectResultType(resType) != RTNORM)  
                return;  
           WriteResult(resType, y);  
      }  
      if(rejectedCount > 0)  
           acutPrintf(_T("\n%i objects rejected."), rejectedCount);  
      acutPrintf(_T("\n%s is %.2f."), addition == 1 ? _T("Addition") :_T("Multiplication"), y);  
 }  
```

```cpp
int TextMath::SelectOperationType(int &addition)  
 {  
      int rc = RTNORM;  
      ACHAR kwOprType[10];   
      acedInitGet(NULL, _T("Add Multiply"));   
      rc = acedGetKword(_T("\nSelect operation type [Add/Multiply]: "), kwOprType);  
      if(rc != RTNORM){  
           if(rc == RTNONE){  
                wcscpy(kwOprType, _T("Add"));  
                rc = RTNORM;  
           }  
           else{  
                acutPrintf(_T("\nUser cancelled."));  
                return rc;  
           }                 
      }  
      if(rc == RTNORM){  
           if (wcscmp(kwOprType, _T("Add")) == 0){  
                addition = 1;  
           }  
           else if (wcscmp(kwOprType, _T("Multiply")) == 0){  
                addition = 0;  
           }  
      }  
      return rc;  
 }  
 int TextMath::SelectResultType(int &result)  
 {  
      int rc = RTNORM;  
      ACHAR kwOprType[10];   
      acedInitGet(NULL, _T("Existing New"));   
      rc = acedGetKword(_T("\nSelect a text object [Existing/New]: "), kwOprType);  
      if(rc != RTNORM){  
           if(rc == RTNONE){  
                wcscpy(kwOprType, _T("Existing"));  
                rc = RTNORM;  
           }  
           else{  
                acutPrintf(_T("\nUser cancelled."));  
                return rc;  
           }                 
      }  
      if(rc == RTNORM){  
           if (wcscmp(kwOprType, _T("Existing")) == 0){  
                result = 1;  
           }  
           else if (wcscmp(kwOprType, _T("New")) == 0){  
                result = 0;  
           }  
      }  
      return rc;  
 }  

```

```cpp
void TextMath::WriteResult(int resultType, double result)  
 {  
      ACHAR strResult[100];  
      if(acdbRToS(result, 2, 2, strResult) != RTNORM)  
           return;  
      if(resultType == 0){  
           ads_point pnt;  
           if(acedGetPoint(NULL, _T("\nSelect a point:"), pnt) != RTNORM)  
                return;  
           AcDbText* text = new AcDbText;  
           text->setDatabaseDefaults();  
           text->setPosition(asPnt3d(pnt));  
           text->setTextString(strResult);  
           TransformToWcs(text, acdbHostApplicationServices()->workingDatabase());  
           AcDbDatabase* pDb = acdbHostApplicationServices()->workingDatabase();  
           AcDbBlockTableRecordPointer pBtr(pDb->currentSpaceId(), AcDb::kForWrite);   
           pBtr->appendAcDbEntity(text);  
           pBtr->close();  
           text->close();            
      }  
      else  
      {  
           ads_name entName;  
           ads_point pnt;  
           AcDbObjectId objId;  
           AcDbEntity *ent = NULL;  
           if(acedEntSel(_T("\nSelect a text/mtext object:"), entName, pnt) != RTNORM)  
                return;  
           if(acdbGetObjectId(objId, entName) != Acad::eOk)  
                return;  
           if(acdbOpenAcDbEntity(ent, objId, AcDb::kForWrite) != Acad::eOk){  
                ent->close();  
                return;  
           }  
           if (ent->isKindOf(AcDbText::desc())) {  
                AcDbText *pText = static_cast<AcDbText*>(ent);  
                pText->setTextString(strResult);       
                pText->close();  
           }  
           else if (ent->isKindOf(AcDbMText::desc())){  
                AcDbMText *pMText = static_cast<AcDbMText*>(ent);  
                pMText->setContents(strResult);  
                pMText->close();  
           }  
           else{  
                acutPrintf(_T("\nNothing selected."));  
           }  
           ent->close();  
      }  
 }  
```

```cpp
void TextMath::TransformToWcs(AcDbEntity* ent, AcDbDatabase* db)  
 {  
   AcGeMatrix3d m;  
      if (!acdbUcsMatrix(m, db)) {  
           m.setToIdentity();  
      }  
   ent->transformBy(m);  
 }  
```
