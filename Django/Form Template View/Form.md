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
form.as_p().as_table().as_ul()
form.clean_data
form.save()
```

It's another HTML generator for data fields, we need to check if it has data, is data valid, get the data after user submitted, and save it to DB.

Still it's a HTML generator :)

- Empty form -> f.is_bound() = False
- data inputed correctly -> f.is_valid() = True
- data -> f.clean_data (a python dict with python values according to field type, request.POST are is a dict of all string values)
- save form -> write it yourself or use ModelForm

**Create a form:**
- `f.__init__()` 
basicly generate HTML from form.
- user request.POST
this is CreateView or UpdateView do
- 

**To be lazy:**


