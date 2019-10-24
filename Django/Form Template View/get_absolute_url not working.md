# Issue - get_absolute_url not working



> Django Class Based Views



I have the same


Solved the problem after read the first sentence: 
```
    def form_valid(self, form):
        article = form.save(commit=False)
        article.author = self.request.user
        self.object = article.save()
        return super().form_valid(form)
```
the point is the save `article.save()`'s result back to self.object

