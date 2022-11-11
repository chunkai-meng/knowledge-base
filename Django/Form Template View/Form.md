# Form

**Some metaphor：**

Take Template as a string
render_to_response() is just a "".format() method

Template is static but render() fill it with python values

Form is a little advance above this.

it has:
```py
form.is_bound()
form.is_valid()
form.as_div() .as_p() .as_table() .as_ul() -> HTML fields
form.render('template.html') -> the HTML form or whatever you want it to look like
form.clean_data
form.save()
```

It's another HTML generator for data fields, we need to check if it has data, is data valid, get the data after user submitted, and save it to DB.

Still it's a HTML generator :)

- Empty form -> f.is_bound() = False
- data inputed correctly -> f.is_valid() = True
- data -> f.clean_data (a python dict with python values according to field type, request.POST are is a dict of all string values)
- save form -> write it yourself or use ModelForm
- `clas Meta:`
```py
    class Meta:
        error_messages = {}
```

**Create a form:**
- `f.__init__(*args, **kwargs)`
 basicly generate HTML from form.
 so Form() is a empty form, Form(request.POST)
 is user posted form. Form(is_vip=request.GET.get('is_vip')) is a dynamic form. 

this is CreateView or UpdateView do
- 

**To be lazy:**


