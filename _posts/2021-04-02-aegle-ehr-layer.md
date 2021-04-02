---
layout: page
title: Aegle - EHR Modernisation Layer

---
## Aegle - EHR Modernisation Layer

At Hack Cambridge 2020 hackathon, my team and I designed a proof of concept for an OCR abstraction layer to bridge legacy Electronic Health Record (EHR) management systems. The app scans EHR webpages or hardcopy images, uses Microsoft Azure’s OCR API to detect key information such as the patient’s name and doctor’s remarks, then stores them in a database. The application was implemented in C#, the database was hosted on Azure. 

{{< youtube kjvqHetWXMc>}}

[My contribution](https://github.com/jeremysee2/Hackbridgetest1) was the C# code that ingested the image and called the Azure Cognitive Services API to generate the JSON output of text in the image. That data was then parsed in Azure Cloud's database, updating the dashboard. Our demonstration of the product can be seen in the video above. During judging and presentations, we received a lot of positive feedback from people who had experienced the pains of losing medical documentation moving from one hospital to the next. In fact, a medical student was very appreciative of our idea - he had spent hours translating paper records into legacy digital ones, and this product would have saved many hours of data entry, focusing healthcare professionials' attention on more important matters.