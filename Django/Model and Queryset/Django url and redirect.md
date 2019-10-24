# Django url and redirect

Created: Aug 01, 2019 11:32 AM
Last Edited Time: Aug 01, 2019 12:00 PM
Status: Completed
Tags: Django

# Path Name → Url

let's say we have urls like below:

    # proj/urls.py
    
    urlpatterns = [
                      path('', admin.site.urls),
                      path('report/purchase/', include('purchase.urls')),
    ]

    # proj/apps/purchase/urls.py
    app_name = 'purchase' # important
    urlpatterns = [
        path('', purchase_list, name='purchase-list'),
        path('suppliers/<int:id>/calculate/', supplier_views.calculate_due, name='supplier-calculate-due'),
    ]

Url of *supplier-calculate-due*:

    from django.urls import reverse
    reverse('purchase:supplier-calculate-due', args=[id])

# Redirect to a custom view

    # option 1: redirect to url directory (not recommand)
    from django.http import HttpResponseRedirect
    from django.shortcuts import redirect
    ....
    	return HttpResponseRedirect('/production/') # or 
    	return HttpResponseRedirect('admin:app_model_change-list')
    
    # option 2: redirect to reversed url
    	url = reverse('production')
    	return redirect(url)

# Redirect to an admin view

    return HttpResponseRedirect('admin:app_model_change', args=[id])

would be rejected for not secure issue.

So use below to workarround:

    	url = reverse('admin:purchase_supplier_change', args=[id])
      return redirect(url + '?add=True')