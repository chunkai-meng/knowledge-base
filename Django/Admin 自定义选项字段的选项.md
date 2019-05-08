# Django populate a form.ChoiceField field from a queryset and relate the choice back to the model object







I run into this question many times when searching by 

'how to customize/populate Django SelectField options'

The answer provided by [Dimitris Kougioumtzis](https://stackoverflow.com/a/44720884/7728887) is quite easy 

Hope it could help somebody like me.

```python
# forms.py
from django.forms import ModelForm, ChoiceField
from .models import MyChoices

class ProjectForm(ModelForm):
    choice = ChoiceField(choices=[
        (choice.pk, choice) for choice in MyChoices.objects.all()])
```

```python
# admin.py
class ProjectAdmin(BaseAdmin):
    form = ProjectForm
    ....
```



[See Also](https://stackoverflow.com/questions/34781524/django-populate-a-form-choicefield-field-from-a-queryset-and-relate-the-choice-b)

