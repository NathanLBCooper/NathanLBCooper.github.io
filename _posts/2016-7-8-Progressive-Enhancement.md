---
layout: post
title: "How to progressively enhance a non-Javascript website"
date: 2016-07-08 12:00:00
excerpt: "I want to improve my website's experience for users with JavaScript, whilst keeping solid non-JavaScript functionality. Remember when you couldn't count on Javascript being there? No, me neither. But not every device is a laptop or modern mobile phone, so let's check this out"
categories: [front-end, programming]
comments: false
image:
  feature: frontend.png
---

I want to improve my website's experience for users with JavaScript, whilst keeping solid non-JavaScript functionality.

I have these goals:

- Improve the experience using Typescript
- Leave the non JavaScript behaviour unchanged
- Leave the MVC code and structure unchanged

My website is a real one, but in this blog post I'll use an imaginary website for keeping track of people's biscuit preferences.

## Biscuit themed example site ##

Here's a very useful table for keeping track of my colleagues biscuit binging habits:

<img src="{{ '/img/PreJsPeopleTable.png' | prepend: site.baseurl }}" alt="Picture of Table before Javascript" style="width: 50%; height: auto;"> 

My site is a traditional HTML-only one. In the event Andy changes his allegiance to custard creams, I'd follow the edit link to an edit form and make my changes there. I don't have multiple forms on this page, which is the tidier options for an HTML page, so there's a bit more navigation than is ideal. 

Here's the table cshtml:

```csharp

@model Biscuits.Models.HomeViewModel
<h2>People</h2>
<table>
    <thead><tr><th>Name</th><th>Favourite Biscuit</th><th></th></tr></thead>
    <tbody>
        @foreach (var person in Model.People)
        {<tr>
            <td>@person.Name</td>
            <td>@person.FavouriteBiscuit.Name</td>
            <td>@Html.ActionLink("Edit", MVC.Home.EditFavouriteBiscuit(person.Id), null)</td>
        </tr>}
    </tbody>
</table>

```

The index returns view with my table, the EditPerson GET returns my edit page, and the EditPerson POST actually does the edit. It's easy and straightforward MVC.

```csharp

[HttpGet]
public virtual ActionResult Index()
{
	...
	return View(model);
}

[HttpGet]
public virtual ActionResult EditPerson(int id)
{
	var person = _personRepo.GetPersonById(id);
	if (person == null)
	{
		return HttpNotFound();
	}

	var biscuits = _biscuitRepo.GetBiscuits();
	var model = new EditPersonModel(person, biscuits);
	return View(model);
}

[HttpPost]
[ValidateAntiForgeryToken]
public virtual ActionResult EditPerson(int id, string newBiscuit)
{
	...
	if (person == null)
	{
		return HttpNotFound();
	}

	...
	
	ModelState.AddErrors(_personRepo.CanEditPerson(person, biscuit));

	if (this.ModelState.IsValid)
	{
		_personRepo.EditPerson(person, biscuit);
		return RedirectToAction("Index");
	}
	
	...
	
	return View(model);
}


```

This works fine. But I'd like for the experience for users with JavaScript enabled to be better.

## The ultimate JavaScipt/Biscuit experience ##

I'd like to edit biscuit preference via drop-downs on one page, and I'd like Save buttons to appear contextually. Something like this:

<img src="{{ '/img/PostJsPeopleTable.png' | prepend: site.baseurl }}" alt="Picture of Table after Javascript" style="width: 50%; height: auto;"> 

The first thing to do is hide the elements I no longer want to see, like the edit links. I'll add this `class="jsenhancement-hide"` to these elements, and hide this class of elements when I run my Typescript.

I'll also need to make the new elements appear, like the biscuit drop-down. I'll create them as initially `hidden` and add this `class="jsenhancement-show hidden"` to them. This time I'll use my Typescript code to reveal the when the page loads. 

Here's my new table. Some of the `<td>` elements, have two types of content in them, one for old experience and one for the shiny JavaScript experience.  I've also added some additional hidden data, like person.Id. This is data I need for my front end code, so I've placed it in the  HTML document.

I didn't add `jsenhancement-show` to my Save button, as I'd still like them to be hidden. I'll be making these buttons appear as the user makes changes.

```csharp

<thead>
    <tr><th>Name</th><th>Favourite Biscuit</th><th></th>
    </tr>
</thead>
<tbody>
    @foreach (var person in Model.People)
    {<tr>
        <td>@person.Name</td>
        <td>
            <span class="jsenhancement-hide">@person.FavouriteBiscuit.Name</span>
            <span class="jsenhancement-show hidden">
                @Html.DropDownList("newBiscuit", Model.GetBiscuitSelectItems(person.FavouriteBiscuit),
                  new { data_jsenhancement = "edit-person-dropdown" })
        </span>
        </td>
        <td>
            @Html.ActionLink("Edit", MVC.Home.EditPerson(person.Id), new { @class = "jsenhancement-hide"}, null)
            <button class="hidden" data-jsenhancement="edit-person-save-button" href="@Url.Action(MVC.Home.EditPerson())">Save</button>
        </td>
        <td data-jsenhancement="person-id" class="hidden">@person.Id</td>
        <td data-jsenhancement="person-originalFavouriteBiscuit" class="hidden">@person.FavouriteBiscuit.Name</td>
    </tr>}
</tbody>

```

Now lets write the Typescript to power this enhancement:

```ts

class Homepage {
    private jsEnhancer: JsEnhancer;
    private people: PeopleTable;

    constructor() {
        this.jsEnhancer = new JsEnhancer();
        this.people = new PeopleTable(this.jsEnhancer);
    }

    public applyJsEnhancements() {

        this.jsEnhancer.hideElements();
        this.jsEnhancer.showElements();

        this.people.applyJsEnhancements();
    }
}

var homepage = new Homepage();
homepage.applyJsEnhancements();

```

`Homepage` is my main entry point, which hides/shows the elements. We need more behaviour than that though, which is contained in my `PeopleTable` class. This is just the code I need to bind the visibility of the save button to the biscuit drop-down and to hook up the save button. Here's that code. There's a lot of messing around with JQuery, but the detail isn't an important thing to grasp.

```ts

class PeopleTable {
    private jsEnhancer: JsEnhancer;

    constructor(jsEnhancer: JsEnhancer) {
		...
    }

    public applyJsEnhancements(): void {
        const dropdowns = document.querySelectorAll(this.dropdownQuery);

        for (let i = 0; i < dropdowns.length; i++) {
			// find the biscuit dropdown, the personId, the Td with the original biscuit value, the save button
			...
			
            this.bindSaveVisibilityToBiscuitDropdown(dropdown, button, originalBiscuitTd);
            this.addPersonSaveHandler(button, dropdown, originalBiscuitTd, personId);
        }
    }

    private addPersonSaveHandler(button: JQuery, dropdown: HTMLSelectElement,
        originalBiscuitTd: JQuery, personId: string): void {
        const self = this;

        button.click(() => {
            const originalOption = originalBiscuitTd.html().toString();
            const selectedOption = $(dropdown).find(":selected").text();
            if (originalOption === selectedOption) {
                return;
            }

            $.ajax({
                type: "POST",
                url: button.attr('href'),
                data: this.jsEnhancer.addAntiForgeryToken({
                    id: personId,
                    newBiscuit: selectedOption
                }),
                success(msg) {
                    var errors = self.jsEnhancer.findValidationErrors(msg);
                    if (errors.length > 0) {
                        alert(errors[0]);
                        return;
                    }

                    originalBiscuitTd.html(selectedOption);
                    button.addClass("hidden");
                },
                error(xhr, ajaxOptions, thrownError) {
                    alert(xhr.status + ": " + thrownError);
                }
            });
        });
    }

    private bindSaveVisibilityToBiscuitDropdown(dropdown: HTMLSelectElement, button: JQuery, originalBiscuitTd: JQuery): void {
        $(dropdown)
            .change(() => {
                const originalOption = originalBiscuitTd.html().toString();
                const selectedOption = $(dropdown).find(":selected").text();

                if (originalOption === selectedOption) {
                    button.addClass("hidden");
                    return;
                } else {
                    button.removeClass("hidden");
                    return;
                }
            });
    }
}

```

And here is the `JsEnhancer` class. This handles showing the 'new' elements and hiding the 'old' ones that belong to the pre-javascript experience. I also handle a few other shared concerns, which I'll explain below.

```ts

class JsEnhancer {
    public hideElements(): void {
        const elements = document.querySelectorAll(".jsenhancement-hide");

        for (let i = 0; i < elements.length; i++) {
            elements[i].classList.add("hidden");
        }
    }

    public showElements(): void {
        const elements = document.querySelectorAll(".jsenhancement-show");

        for (let i = 0; i < elements.length; i++) {
            elements[i].classList.remove("hidden");
        }
    }

    // Find validation errors in a page with a Validation Summary
    public findValidationErrors(msg: string): string[] {
        const summaryElement = $(msg).find("#validation-summary");
        var items = $(summaryElement).find("li");

        let errors = [];
        for (let i = 0; i < items.length; ++i) {
            errors.push(items[i].innerHTML.toString());
        }

        return errors;
    }

    // Use antiforgery token in layout page for an ajax request
    addAntiForgeryToken(data) {
        data.__RequestVerificationToken = $("#__AjaxAntiForgeryForm input[name=__RequestVerificationToken]").val();
        return data;
    };
}

```

**findValidationErrors:**

Digs the validation message out of the View returned by a POST request. This is the bit I'd change if I did it again. 

For example, `EditPerson`, in the case of ModelState errors, returns the Edit View with some elements in my Validation Summary:

`@Html.ValidationSummary(false, String.Empty, new { id = "validation-summary" })`

My ajax request has 'succeeded' but I need to find and read this Validation Summary to see if my changes actually got made.

I'm not happy with this, it's it a very weak link between my front end code and my back end code. My error handling is pretty basic right now. I think this weak foundation will cause problems if I want to do something more elaborate. I suspect I'll have to modify my controllers to accommodate the Ajax requests. On the other hand, I'll achieved my goal of creating a JavaScript experience without changing any actual code. But maybe that wasn't a very sensible goal?


**addAntiForgeryToken:**

This is a trick to make the antiforgery tokens work with Ajax requests. Read about it in the StackOverflow question  [jQuery Ajax calls and the Html.AntiForgeryToken()](http://stackoverflow.com/a/4074289/1734730).

It uses a form I've created in the _Layout.cshtml:

```csharp

<form id="__AjaxAntiForgeryForm" action="#" method="post">
    @Html.AntiForgeryToken()
</form>	

```


## Summary ##

So my biscuit website now provides a better experience with JavaScript. The non-Javascript experience is unchanged.

The error handling isn't quite good enough yet, but I like that I've been able to manage so far with a short Typescript file and a few simple changes to the cshtml.

For the record, I don't think the need to support non-JavaScript browsers exists nowadays, even on phones. But it's nice to know it's not that hard to extend plain html apps, to tack on key bits of functionality, without entirely re-writing them.