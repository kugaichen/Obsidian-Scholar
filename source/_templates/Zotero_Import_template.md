---
type: paper
title: "{{ title }}"
publicationTitle: "{{ publicationTitle | default(proceedingsTitle) | default(conferenceName) | default('') | replace(r/.*SIGCOMM.*/i, '#SIGCOMM') | replace(r/.*NSDI.*/i, '#NSDI') | replace(r/.*ISCA.*/i, '#ISCA') | replace(r/.*OSDI.*/i, '#OSDI') | replace(r/.*ATC.*/i, '#ATC') | replace(r/.*EuroSys.*/i, '#EuroSys') | replace(r/.*FAST.*/i, '#FAST') | replace(r/.*SOSP.*/i, '#SOSP') | replace(r/.*MICRO.*/i, '#MICRO') | replace(r/.*ASPLOS.*/i, '#ASPLOS') | replace(r/.*HPCA.*/i, '#HPCA') | replace(r/.*INFOCOM.*/i, '#INFOCOM') | replace(r/.*CoNEXT.*/i, '#CoNEXT') | replace(r/.*MobiCom.*/i, '#MobiCom') | replace(r/.*NeurIPS.*/i, '#NeurIPS') | replace(r/.*ICML.*/i, '#ICML') | replace(r/.*ICLR.*/i, '#ICLR') | replace(r/.*CVPR.*/i, '#CVPR') | replace(r/.*ICCV.*/i, '#ICCV') | replace(r/.*ECCV.*/i, '#ECCV') | replace(r/.*IEEE Transactions on.*/i, '#Trans') | replace(r/.*ACM Transactions on.*/i, '#Trans') }}"
year: "{{ date | format('YYYY') }}"
doi: "{{ DOI }}"
authors: ["{{ creators[0].firstName }} {{ creators[0].lastName }}"]
topics: [{% for tag in tags %}"#{{ tag.tag }}"{% if not loop.last %}, {% endif %}{% endfor %}]
tags: [{% for tag in tags %}"#{{ tag.tag }}"{% if not loop.last %}, {% endif %}{% endfor %}, "#未读"]
status: "#未读"
zoteroLink: "{{ select }}"
created: "{{ date | format('YYYY-MM-DD') }}"
---

# {{ title }}

## 一句话总结
oneSentence:: 

---

## 表格关键字段
**优点 (Pros):**
- pros:: 

**缺点 (Cons):**
- cons:: 

**感想 (Thoughts):**
- thoughts:: 

---

## 详细笔记
### 关键点 (Key Points)
- 

### 方法 / 公式 (Method)
- 

### 结果 (Results)
- 

### 个人评论 / 可复现性
- 
--- 

## 引用信息
- **DOI**: [{{ DOI }}](https://doi.org/{{ DOI }})
- **期刊/会议**: {{ publicationTitle | default(proceedingsTitle) | default(conferenceName) }}
- **Zotero Link**: [Open in Zotero]({{ select }})
- **PDF Link**: [Open PDF]({% for attachment in attachments | filterby('contentType', 'application/pdf') %}file:///{{ attachment.path | replace(" ", "%20") }}{% endfor %})

---
## Zotero 注释导入

{% persist "annotations" %}
{% set annotations = annotations | filterby("date", "dateafter", lastImportDate) -%}
{% if annotations.length > 0 %}
### 新导入的注释 ({{ importDate | format("YYYY-MM-DD HH:mm") }})

{% for annotation in annotations %}
{% if annotation.imageRelativePath %}
![[annotation.imageRelativePath]]
{% endif %}
{% if annotation.comment %}
> **笔记**: {{ annotation.comment }}
{% endif %}
> {{ annotation.annotatedText }} [p. {{ annotation.pageLabel }}](zotero://open-pdf/library/items/{{ annotation.attachment.itemKey }}?page={{ annotation.pageLabel }}&annotation={{ annotation.id }})
{% endfor %}
{% endif %}
{% endpersist %}