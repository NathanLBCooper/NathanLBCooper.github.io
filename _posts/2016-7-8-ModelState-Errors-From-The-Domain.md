---
layout: post
title: "Passing domain errors into your model state"
date: 2016-07-08 12:00:00
excerpt: "Pass your domain errors to your caller without boilerplate"
categories: [how-to, programming, asp.net]
comments: false
image:
  feature: code.png
---

Let's say I'm writing a web app to add People to a GuestList. The GuestList is a sensible place to keep the logic to decide whether a person is added and the reason why. For example:

```csharp

public class BobHatingGuestList : IGuestList
{

...

public IEnumerable<ValidationResult>(Person person)
{

    var errors = new List<ValidationResult>();

    if (person.Name.Equals("bob", StringComparison.InvariantCultureIgnoreCase));
    {
        errors.Add(new ValidationResult("I really hate all bobs", new[] { "person" }));
        return errors;
    }

    if (_guestCount >= _maxGuestCount)
    {
        errors.Add(new ValidationResult("The list is full", new[] { "person" }));
        return errors;
    }

	return errors; 
}

}

```

We can just add this to the ModelState:

```csharp

[HttpPost]
public ActionResult AddPerson(id personId, id guestListId){

	...
	
	ModelState.AddErrors(_guestList.CanAddPerson(person));
	
	if (this.ModelState.IsValid)
	{
	    _guestList.Add(person);
	    return RedirectToAction("Index");
	}
	
	...

	return View(model);
}

```

Using this extension method

```csharp

public static class ModelStateDictionaryExtensions
{
    public static void AddErrors(this ModelStateDictionary modelStateDictionary,
        IEnumerable<ValidationResult> validationResults)
    {
        if (validationResults == null)
            return;

        foreach (var validationResult in validationResults)
        {
            if (validationResult.MemberNames.Any())
            {
                foreach (var member in validationResult.MemberNames)
                {
                    modelStateDictionary.AddModelError(member, validationResult.ErrorMessage);
                }
            }
            else
                modelStateDictionary.AddModelError(String.Empty, validationResult.ErrorMessage);
        }
    }
}

```
