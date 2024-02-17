<h1 align="center">evropidan.ge</h1>

A simple landing webpage project on Django for a postal company. The idea is that in order to obtain the shipping address, the user must register and, after confirming their email address, he/she will receive the shipping address.

[Deployed Webpage](https://www.evropidan.ge)

#### Technologies:
- Django 4.2.7
- Django Allauth
- Django REST Framework
- Bootstrap 5.3.2
- Python
- JavaScript
- HTML/CSS
- Git
- GitHub


#### Deployment:
The project is deployed on an Ubuntu server on [Linode](https://www.linode.com). 

###### Deployment Steps:
- Creating a Linux server.
- Setting the hostname.
- Adding a limited user account.
- Setting up SSH authentication.
- Installing and setting up UFW (Uncomplicated Firewall).
- Copying the project to the server.
- Installing dependencies.
- Installing Python.
- Setting up the virtual environment.
- Installing and configuring Apache2.
- Setting up permissions.
- Enabling HTTPS using Let's Encrypt.

Due to the project's private business nature, I'm sharing selected code snippets to demonstrate my coding skills. 

###### <span id="list-of-file">List of Files</span>

-   [Home page views.py](#home-views)
-   [Example of the template file](#html-example)

### <span id="home-views"> Home page views.py</span>

```python
# Django imports.
from django.core.paginator import Paginator
from django.http.response import HttpResponse as HttpResponse
from django.shortcuts import render, redirect
from django.urls import reverse
from django.views import View

# Third-party imports.
from allauth.account.models import EmailAddress

# Local application imports.
from home.models import Question, ShopList


class HomeView(View):
    """
    Home page view.
    """
    template_name = 'home/index.html'

    def dispatch(self, request, *args, **kwargs):      
        # If the user is authenticated, check if they have confirmed their email.
        if request.user.is_authenticated:
            email_confirmed = EmailAddress.objects.filter(
                user=request.user, verified=True).exists()

            # If the user has not confirmed their email, redirect to the verification page.
            if not email_confirmed:
                return redirect(reverse('account_email_verification_sent'))
        
        return super().dispatch(request, *args, **kwargs)
    

    # Overide the get method.
    def get (self, request):
        # Grab the questions.
        questions = Question.objects.all().order_by('order')
        # Grab the shop lists.
        shop_lists = ShopList.objects.all()
        
        # Paginate the shop lists.
        paginator = Paginator(shop_lists, 20)
        page_number = self.request.GET.get('page')
        shops = paginator.get_page(page_number)

        # Return the rendered template.
        return render(request, 
                      self.template_name,
                      {
                          'questions': questions,
                          'shop_lists': shops
                      }
                    )
```


### <span id="html-example"> The home page hero section's html. </span>
<a href="#list-of-file">
    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-up-square" viewBox="0 0 16 16">
    <path fill-rule="evenodd" d="M15 2a1 1 0 0 0-1-1H2a1 1 0 0 0-1 1v12a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1zM0 2a2 2 0 0 1 2-2h12a2 2 0 0 1 2 2v12a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2zm8.5 9.5a.5.5 0 0 1-1 0V5.707L5.354 7.854a.5.5 0 1 1-.708-.708l3-3a.5.5 0 0 1 .708 0l3 3a.5.5 0 0 1-.708.708L8.5 5.707z"/>
    </svg>
</a>

To improve the code's readability and structure, I created a custom template tag to replace the standard each HTML element.

###### HTML example.
```python
<!-- Desktop menu. -->
{% nav 'd-none d-lg-block desktop-menu desktop-menu-anime top-menu' %}
    {% ul %}
        {% li %}
            {% a '#hero-section' 'active' '' 'section:hero-section' %} მთავარი {% enda  %}
        {% endli %}
        {% li %}
            {% a '#price-section' '' '' 'section:price-section' %} ღირებულება {% enda %}
        {% endli %}
        {% li %}
            {% a '#shop-section' '' '' 'section:shop-section' %} მაღაზიები {% enda %}
        {% endli %}
        {% li %}
            {% a '#qa-section' '' '' 'section:qa-section' %} კითხვა-პასუხი {% enda %}
        {% endli %}
    {% endul %}
{% endnav %}
```
###### Template tag.
```python
# <nav> element
@register.simple_tag
def nav(class_attr='', id_attr=''):
    return format_html("<nav class='{}' id='{}'>", class_attr, id_attr)

@register.simple_tag
def endnav():
    return format_html("</nav>")

# <a> element
@register.simple_tag
def a(href='', cls='', id='', data_attr=''):
    if data_attr:
        data_split = data_attr.split(':')
        data_attr = f'data-{data_split[0]}={data_split[1]}'

    if href and href[0] != '#':
        href = reverse(href)

    return format_html('<a href="{}" class="{}" id="{}" {}>', href, cls, id, data_attr)

@register.simple_tag
def enda():
    return format_html("</a>")
    ...
```


