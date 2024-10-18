## [OSINT] The Investigator
```
Help Jieyab found the newspaper. When was this newspaper published?

The flag name is date TCP1P{Date Month Year}

Example: TCP1P{29 January 2001}
```

新聞記事が渡される。  
![](assets/PETRUS%20roeit%20Indonesische%20misdaad%20uit.png)

記事内の文章を片っ端から検索かけても見つからなかったので、アーカイブから記事を探すことにした。  
オランダ語なのでオランダの新聞だろうとあたりをつけて、 [Wikipedia:List of online newspaper archives](https://en.wikipedia.org/wiki/Wikipedia:List_of_online_newspaper_archives#Netherlands) のオランダのサイトを上から見て行った。  

２行目のサイトで `DE FANATIEKE MISSIE VAN Zia ul Haq` で検索したところ発見。  
source: [De Telegraaf » 17 dec 1983 - Art. 728 | Delpher](https://www.delpher.nl/nl/kranten/view?query=DE+FANATIEKE+MISSIE+VAN+Zia+ul+Haq&coll=ddd&identifier=ddd:011205843:mpeg21:a0728&resultsidentifier=ddd:011205843:mpeg21:a0728&rowid=1)  

`TCP1P{17 December 1983}`