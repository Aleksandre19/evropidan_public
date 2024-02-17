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

### <span id="list-of-file">List of Files</span>
Due to the project's private business nature, I'm sharing selected code snippets to demonstrate my coding skills.

-   [Home page views.py](#home-views)
-   [HTML Structuring](#html-example)
-   [Menu](#menu)
    -   [Underline on hover](#underline-on-hover)
    -   [Mobile menu](#mobile-menu)

### <span id="home-views"> Home page views.py</span>
<small> <a href="#list-of-file"> Menu ^ </a> </small>

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


### <span id="html-example"> HTML Structuring </span>

To improve the code's readability and structure, I created a custom template tag to replace the standard each HTML element.
<small> <a href="#list-of-file"> Menu ^ </a> </small>

###### HTML Example:
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
###### Template Tags Example:
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

### <span id="menu">Menu</span>

#### <span id="underline-on-hover">Underline on hover</span>
This is the code for the hover underline effect on the desktop top menu.
<small> <a href="#list-of-file"> Menu ^ </a> </small>

###### Underline HTML:
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
###### Underline CSS:
```css
/* Menu link underline on hover. */
.hero-section nav.desktop-menu ul li a::after {
    content: "";
    position: absolute;
    bottom: -3px;
    left: 50%;
    width: 0;
    height: 2px;
    background-color: var(--main-color);
    transition: width 0.3s ease-in-out, left 0.3s ease-in-out;
}

/* When link is being hoverd. */
.hero-section nav.desktop-menu ul li a:hover::after {
    width: 100%;
    left: 0;
}
```

### Mobile menu.
Below are the code snippets divided into sections for the mobile menu.
<small><a href="#list-of-file"> Menu ^ </a> </small>

##### <span id="mobile-menu-sections"> Mobile Menu Sections: </span>
-   [Mobile Menu HTML](#mobile-menu-html)
-   [Mobile Menu CSS](#mobile-menu-css)
-   [Mobile Menu JavaScript](#mobile-menu-js)

###### <span id="mobile-menu-html"> Mobile Menu HTML: </span>
<small><a href="#mobile-menu-sections"> Mobile  Menu Sections ^ </a></small>

```python
<!-- Menu on mobile & tablet devices. -->
{% nav 'd-lg-none mobile-menu top-menu' %}

    <!-- Toggle button. -->
    {% input 'checkbox' 'toggler mobile-toggler' %}
    
    <!-- Hamburger icon. -->
    {% div 'hamburger' %}{% div %}{% enddiv %}{% enddiv %}
    
    <!-- Menu. -->
    {% div 'menu' %}
        {% div %}
            {% div %}
                {% ul %}
                    {% li %}
                        {% a '#hero-section' 'active' '' 'section:hero-section' %} მთავარი {% enda%}
                    {% endli %}
                    {% li %}
                        {% a '#price-section' '' '' 'section:price-section' %} ღირებულება {% enda %}
                    {% endli %}
                    {% li %}
                        {% a '#shop-section' '' '' 'section:shop-section' %} მაღაზიები {% enda %}
                    {% endli %}
                    {% li %}
                        {% a '#qa-section' '' '' 'section:qa' %} კითხვა-პასუხი {% enda %}
                    {% endli %}
                {% endul %}
                {% endul %}
            {% enddiv %}
        {% enddiv %}
    {% enddiv %}
{%  endnav %}
```

###### <span id="mobile-menu-css"> Mobile Menu CSS: </span>
<small><a href="#mobile-menu-sections"> Mobile  Menu Sections ^ </a></small>

```css
/* =============== Mobile menu =========== */
    .mobile-menu {
        position: fixed;
        top: 32px;
        right: 22px;
        width: 62px;
        height: 62px;
        z-index: 100;
    }

    .mobile-menu .toggler {
        position: absolute;
        top: 0;
        left: 0;
        z-index: 2;
        cursor: pointer;
        width: 50px;
        height: 50px;
        opacity: 0;
    }

    .mobile-menu .hamburger {
        position: absolute;
        top: 0;
        left: 0;
        z-index: 1;
        width: 60px;
        height: 60px;
        border-radius: 50%;
        padding: 1rem;
        box-shadow: 4px 4px 10px rgb(0 0 0 / 24%);
        background: var(--main-color-hover);
        display: flex;
        align-items: center;
        justify-content: center;
        transition: all 0.3s ease-in-out;
    }

    /* Hamburger Line */
    .mobile-menu .hamburger>div {
        position: relative;
        flex: none;
        width: 100%;
        height: 2px;
        background: var(--main-dark);
        display: flex;
        align-items: center;
        justify-content: center;
        transition: all 0.4s ease;
    }

    /* Hamburger Lines - Top & Bottom */
    .mobile-menu .hamburger>div::before,
    .mobile-menu .hamburger>div::after {
        content: '';
        position: absolute;
        z-index: 1;
        top: -10px;
        width: 100%;
        height: 2px;
        background: inherit;
    }

    /* Moves Line Down */
    .mobile-menu .hamburger>div::after {
        top: 10px;
    }

    .mobile-menu .toggler:checked+.hamburger {
        background-color: var(--main-dark);
    }

    /* Toggler Animation */
    .mobile-menu .toggler:checked+.hamburger>div {
        background: var(--main-color-hover);
        transform: rotate(135deg);
    }

    /* Turns Lines Into X */
    .mobile-menu .toggler:checked+.hamburger>div:before,
    .mobile-menu .toggler:checked+.hamburger>div:after {
        top: 0;
        transform: rotate(90deg);
    }

    .mobile-menu .toggler:checked~.menu {
        visibility: visible;
    }


    .mobile-menu .toggler:checked~.menu>div {
        transform: scale(1);
        transition-duration: 0.75s;
    }

    .mobile-menu .toggler:checked~.menu>div>div {
        opacity: 1;
        transition: opacity 0.4s ease 0.4s;
    }

    .mobile-menu .menu {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        visibility: hidden;
        overflow: hidden;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .mobile-menu .menu>div {
        background: var(--main-color-hover);
        border-radius: 50%;
        width: 300vw;
        height: 300vw;
        display: flex;
        flex: none;
        align-items: center;
        justify-content: center;
        transform: scale(0);
        transition: all 0.4s ease;
    }

    .mobile-menu .menu>div>div {
        text-align: center;
        max-width: 90vw;
        max-height: 100vh;
        opacity: 0;
        transition: opacity 0.4s ease;
    }

    .mobile-menu .menu ul {
        padding-left: 0;
    }

    .mobile-menu .menu>div>div>ul>li {
        list-style: none;
        color: #fff;
        font-size: 1.5rem;
        padding: 1rem;
    }

    .mobile-menu .menu>div>div>ul>li>a {
        font-family: 'Heading';
        font-size: 32px;
        letter-spacing: 1.5px;
        color: var(--main-dark);
        text-decoration: none;
        margin: 8px 24px;
    }
```
###### <span id="mobile-menu-js"> JavaScript: </span>
These code snippets are located within the TopMenu JavaScript class, in the onclick event function of the menu links, and it causes the menu to disappear halfway when the link is clicked.
<small><a href="#mobile-menu-sections"> Mobile  Menu Sections ^ </a></small>

```javascript
// Calculate offset fot scrolling.
const offset = 120; // Margin from the top.
const rect = section.getBoundingClientRect();
const winPageOff = window.scrollY;

// Calculete the section current position.
const offsetPosition = (rect.top + winPageOff) - offset;

// Offset for mobile and tablet devices.
const mobileOffset = offsetPosition * 0.8;
```
```javascript
// Set interval to check if the page has been scrolled.
// If so, adding underline and color to the current link.
const checkScroll = setInterval(()=> {

    // In case of mobile and tablet devices the opened menu
    // disappeares half way on the scrolling to the section. 
    if (window.innerWidth < 992){
        this.closeMobileMenu(offsetPosition, mobileOffset, offset);
    }
}, 100);
```
```javascript
// Close the opened mobile menu on the scrolling to the section.
closeMobileMenu(offsetPosition, mobileOffset, offset) {
    if (window.scrollY - offsetPosition - offset <= (mobileOffset + offset) && 
        window.scrollY - offsetPosition - offset >= -(mobileOffset + offset) ) {
            const checkElm = document.querySelector('.mobile-toggler');
            checkElm.checked = false;                           
    }
}
```


