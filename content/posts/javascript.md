---
title: "i18n - vanilla javascript"
date: 2017-03-08T16:38:47Z
draft: true
---

I am getting tired of all this javascript frameworks.

Some years ago it was almost impossible to maintain a webapp compatible to all relevant browsers without some framework like jquery.

Now browsers have evolved enough to make it optional. And given the choice, I prefer the lightweight simpler approach of using vanilla js.

It is amazing how some simple features are usually done with plugins. One of them is translation.
Ok, if you need more extensive support, it might be a good idea to use something but for the basics there is no need for anything more than a json file and one command

Example:
{{< highlight javascript >}}
var i18n_en = {
	weekdays: ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"],  
	short_weekdays: ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
	months: ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"],
}
var i18n_pt = {
	weekdays: ["Segunda", "Terça", "Quarta", "Quinta", "Sexta", "Sábado", "Domingo"],
	short_weekdays: ["Seg", "Ter", "Qua", "Qui", "Sex", "Sáb", "Dom"],
	months: ["Janeiro", "Fevereiro", "Março", "Abril", "Maio", "Junho", "Julho", "Agosto", "Setembro", "Outubro", "Novembro", "Dezembro"],
}
{{< / highlight >}}

{{< highlight html >}}
<button id="butLoginCancel" class="mdc-button translate">but_cancel</button>
{{< / highlight >}}
{{< highlight javascript >}}
for(var el of document.querySelectorAll(".translate")){
    if(i18n[el.innerText])
      el.innerHTML = i18n[el.innerText.toLowerCase()];
}
{{< / highlight >}}

So here you have it, it will replace all the elements with the .translate class and a content matching your translation json

Ah, and to use the i18n, you just have to load that variable with the selected version:
{{< highlight javascript >}}
var i18n = lang=="pt" ? i18n_pt : i18n_en
{{< / highlight >}}

that is it for now
