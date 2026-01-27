
```dataview  
TABLE WITHOUT ID
    file.link as "File",
    choice(zoteroLink, "[" + default(title, "未命名") + "](" + zoteroLink + ")", default(title, "未命名")) as "标题",
    year as "时间",
    publicationTitle as "会议",
    topics as "主题",
    status as "状态",
    oneSentence as "一句话描述",
    choice(pros, "<ul>" + join(map(flat(list(split(string(pros), "[;；]"))), (x) => "<li>" + x + "</li>"), "") + "</ul>", "") as "优点",
    choice(cons, "<ul>" + join(map(flat(list(split(string(cons), "[;；]"))), (x) => "<li>" + x + "</li>"), "") + "</ul>", "") as "缺点",
    choice(thoughts, "<ul>" + join(map(flat(list(split(string(thoughts), "[;；]"))), (x) => "<li>" + x + "</li>"), "") + "</ul>", "") as "感想"
FROM "001_Paper_note_in"
WHERE type = "paper"
SORT year DESC
```

