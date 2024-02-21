<h1 align="center">evropidan.ge</h1>

A simple landing webpage project on Django for a postal company. The idea is that in order to obtain the shipping address, the user must register and, after confirming their email address, he/she will receive the shipping address.

[Deployed Webpage](https://www.evropidan.ge)


### <span id="list-of-file">List of Files</span>
Due to the project's private business nature, I'm sharing selected code snippets to demonstrate my coding skills.

-   [Home Page Views.py](#home-views)
-   [HTML Structuring](#html-example)
-   [Menu](#menu)
    -   [Underline on Hover](#underline-on-hover)
    -   [Mobile Menu](#mobile-menu)
    -   [Mobile Menu JavaScript](#mobile-menu-js-func)
-   [How to Use Cards](#how-to-use-cards)
-   [Custom User Model](#custom-user-model)
-   [Smooth Appearance of Elements on Scrolling - JavaScript](#on-scrolling)
-   [Load the Shop List - JavsScript](#load-shop-list)
-   [Send the Email](#send-email)


#### Technologies:
- Django 4.2.7
- Django Allauth
- Django REST Framework
- Bootstrap 5.3.2
- Python 3.9.1
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
- Enabling HTTPS using Let's Encryp

### <span id="home-views"> Home page views.py</span>
*<a href="#list-of-file"> Back to files list ^ </a>*

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

To improve the code's readability and structure, I created a custom template tag to replace the standard each HTML element. </br>
*<a href="#list-of-file"> Back to files lists ^ </a>*

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
</br>
*<a href="#list-of-file"> Back to files list ^ </a>*

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
Below are the code snippets divided into sections for the mobile menu.</br>
*<a href="#list-of-file"> Back to files list ^ </a>*

##### <span id="mobile-menu-sections"> Mobile Menu Section Links: </span>
-   [Mobile Menu HTML](#mobile-menu-html)
-   [Mobile Menu CSS](#mobile-menu-css)
-   [Mobile Menu JavaScript](#mobile-menu-js)

###### <span id="mobile-menu-html"> Mobile Menu HTML: </span>
*<a href="#mobile-menu-sections"> Back to links ^ </a>*

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
*<a href="#mobile-menu-sections"> Back to links ^ </a>*

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
These code snippets are located within the TopMenu JavaScript class, in the onclick event function of the menu links, and it causes the menu to disappear halfway when the link is clicked.</br>
*<a href="#mobile-menu-sections"> Back to links ^ </a>*

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

### <span id="mobile-menu-js-func">Mobile menu JavaScript functionality.</span>
The following code snippet makes the page scroll to the section when the menu link is clicked. </br>
*<a href="#list-of-file"> Back to files list ^ </a>*
```javascript
// Top menu.
class TopMenu {
    constructor() {
        this.scrolled = false;
        this.runProcess();
    }

    runProcess() {
        this.grabElements(); // Grab necessary elements.
        this.setEvenetToLinks;
    }

    grabElements() {
        this.menuElm = document.querySelector('.menu-wrapper');
        this.menuLinks = document.querySelectorAll('.top-menu a');
    }

    get setEvenetToLinks() {
        this.menuLinks.forEach(link => {
            const sectionID = link.getAttribute('data-section');

            link.addEventListener('click', (e) => {
                e.preventDefault();
                
                // Scroll the page to the current section
                // and style the menu link accordingly.
                this.scrollPage(sectionID, link);
                
            })
        });
    }

    scrollPage(sectionID, link){

        if(sectionID){
            // Select section.
            const section = document.getElementById(`${sectionID}`);

            // Calculate offset fot scrolling.
            const offset = 120; // Margin from the top.
            const rect = section.getBoundingClientRect();
            const winPageOff = window.scrollY;

            // Calculete the section current position.
            const offsetPosition = (rect.top + winPageOff) - offset;

            // Offset for mobile and tablet devices.
            const mobileOffset = offsetPosition * 0.8;
            
            // Scroll window.
            window.scrollTo({
                top: offsetPosition,
                behavior: 'smooth',
            });

            // Set interval to check if the page has been scrolled.
            // If so, adding underline and color to the current link.
            const checkScroll = setInterval(()=> {

                // In case of mobile and tablet devices the opened menu
                // disappeares half way on the scrolling to the section. 
                if (window.innerWidth < 992){
                    this.closeMobileMenu(offsetPosition, mobileOffset, offset);
                }

                // If the page has been scrolled. 
                if (window.scrollY - offsetPosition <= offset && 
                    window.scrollY - offsetPosition >= -offset ) {
                    
                        // Remove active class.
                    this.menuLinks.forEach(l => {

                        l.style.color = '#dcf1b4';

                        if (window.innerWidth < 992)
                            l.style.color = '#000000';

                        l.classList.remove('active');
                    })
                    
                    link.classList.add('active'); // Add active class.
                    link.style.color = '#d5f891'; // Change link color.

                    clearInterval(checkScroll);
                }
            }, 100);
        }
    }

    // Close the opened mobile menu on the scrolling to the section.
    closeMobileMenu(offsetPosition, mobileOffset, offset) {
        if (window.scrollY - offsetPosition - offset <= (mobileOffset + offset) && 
            window.scrollY - offsetPosition - offset >= -(mobileOffset + offset) ) {
                const checkElm = document.querySelector('.mobile-toggler');
                checkElm.checked = false;                           
        }
    }

    // When the page is scrolled more than 200px 
    // adjust it's styles and animate it.
    visibility() {
        if( window.scrollY > 200 ) {
            this.menuElm.classList.remove('top');

            if(!this.scrolled) {
                this.menuElm.style.transform = 'translateY(-200px)';
            }

            setTimeout(()=> {
                this.menuElm.style.transform = 'translateY(0)';
                this.scrolled = true;
            }, 200);

        } else {
            this.menuElm.classList.add('top');
            this.scrolled = false;
        }

    }

}


// Initialize. 
const topMenu = new TopMenu();
```

### <span id="how-to-use-cards"> How To Use Cards </span>
This code implements the sliding card effect in the 'How To Use' section. The cards slide in from the left as the user scrolls down.</br>
*<a href="#list-of-file"> Back to files list ^ </a>*

##### <span id="how-to-use-cards-section"> How To Use Cards Section Links: </span>
-   [How To Use Cards HTML](#how-to-use-html)
-   [How To Use Cards CSS](#how-to-use-css)

###### <span id="how-to-use-html"> How To Use Cards HTML: </span>
*<a href="#how-to-use-cards-section"> Back to links ^ </a>*

```python
<!-- List of steps. -->
{% ul 'how-to-use-list' %}
    <!-- Step 1. -->
    {% li %}
        {% div 'single-task' %}
            {% h3 'task-title' %}
                გაიარეთ რეგისტრაცია
            {% endh3 %}
            {% p 'task-desc' %}
                დარეგისტრირდით ჩვენ ვებ-გვერდზე სულ რამდენიმე ველის შევსებით.
            {% endp %}
        {% enddiv %}
    {% endli %}

    <!-- Step 2. -->
    {% li %}
        {% div 'single-task' %}
            {% h3 'task-title' %}
                დაადასტურეთ ელ.ფოსტა
            {% endh3 %}
            {% p 'task-desc' %}
                რეგისტრაციის შემდეგ მითითებულ ელ.ფოსტაზე მიიღებთ ინსტრუქციას
                ელ.ფოსტის დადასტურების შესახებ.
            {% endp %}
        {% enddiv %}
    {% endli %}
    
    <!-- Step 3. -->
    {% li %}
        {% div 'single-task' %}
            {% h3 'task-title' %}
                მიიღეთ მისამართი
            {% endh3 %}
            {% p 'task-desc' %}
                ელ.ფოსტის დადასტურების შემდეგ მეილზე მიიღებთ მისამართს.
            {% endp %}
        {% enddiv %}
    {% endli %}
    
    <!-- Step 4. -->
    {% li %}
        {% div 'single-task' %}
            {% h3 'task-title' %}
                მიუთითეთ მისამართი
            {% endh3 %}
            {% p 'task-desc' %}
                თქვენთვის სასურველ ონლაინ მაღაზიაში შეიძინეთ სასურველი ნივთი
                და მიმღების გრაფაში მიუთითეთ ელ.ფოსტაზე მიღებული მისამართი.
            {% endp %}
        {% enddiv %}
    {% endli %}
    
    <!-- Step 5. -->
    {% li %}
        {% div 'single-task' %}
            {% h3 'task-title' %}
                დაარეგისტრირეთ გზავნილი
            {% endh3 %}
            {% p 'task-desc' %}
                შეძენის შემდეგ მობრძანდით ჩვენ ვებ-გვერდზე, გაიარეთ ავტორიზაცია
                და დაარეგისტრირეთ გზავნილი.
            {% endp %}
        {% enddiv %}
    {% endli %}
    
    <!-- Step 6. -->
    {% li %}
        {% div 'single-task' %}
            {% h3 'task-title' %}
                ადევნეთ თვალი გზავნილს
            {% endh3 %}
            {% p 'task-desc' %}
                გზავნილის დარეგისტრირების შემდეგ საშუალება გექნებათ თვალი
                ადევნოთ გზავნილის სტატუსს.
            {% endp %}
        {% enddiv %}
    {% endli %}
{% endul %}
```

###### <span id="how-to-use-css"> How To Use Cards CSS: </span>
*<a href="#how-to-use-cards-section"> Back to links ^ </a>*


```css
.how-to-use .how-to-use-list {
    padding: 64px 0;
}

.how-to-use .how-to-use-list li {
    list-style: none;
    position: relative;
    width: 2px;
    margin: 0;
    padding: 22px 0;
    background-color: var(--main-dark);
}

.how-to-use .how-to-use-list li:nth-child(even) .single-task {
    left: 64px;
}


.how-to-use .how-to-use-list li:nth-child(odd) .single-task {
    left: 128px;
}

.how-to-use .how-to-use-list li::after {
    content: '';
    position: absolute;
    left: 50%;
    bottom: 22px;
    width: 45px;
    height: 45px;
    background-color: var(--main-color-hover);
    transform: translateX(-50%);
    border-radius: 50%;
    background-repeat: no-repeat;
    background-position: center;
    background-size: 32px 32px;
    transition: background 0.5s ease-in-out;
}

/* Registration icon. */
.how-to-use .how-to-use-list li:nth-child(1)::after {
    background-image: url('../../images/icons/user-transparant.png');

}

/* Email confirmation icon. */
.how-to-use .how-to-use-list li:nth-child(2)::after {
    background-image: url('../../images/icons/at-transparant.png');
}

/* Address icon. */
.how-to-use .how-to-use-list li:nth-child(3)::after {
    background-image: url('../../images/icons/street-transparant.png');
}

/* Shopping icon. */
.how-to-use .how-to-use-list li:nth-child(4)::after {
    background-image: url('../../images/icons/shopping-transparant.png');
}

/* Add parcel icon. */
.how-to-use .how-to-use-list li:nth-child(5)::after {
    background-image: url('../../images/icons/edit-transparant.png');
}

/* Checking parcel icon. */
.how-to-use .how-to-use-list li:nth-child(6)::after {
    background-image: url('../../images/icons/heart-eyes-transparant.png');
}

.price-section .how-to-use-list .single-task,
.how-to-use .how-to-use-list .single-task {
    position: relative;
    width: 700px;
    border-left: 3px solid var(--main-dark);
    border-radius: 8px;
    padding: 8px 24px;
    background-color: rgba(255, 255, 255, 0.2);
    box-shadow: 3px 3px 10px rgb(0 0 0 / 10%);
    transform: translateX(200px);
    visibility: hidden;
    opacity: 0;
    filter: blur(10px);
    transition: all 0.5s ease-in-out;
}

.how-to-use .how-to-use-list .single-task::before {
    content: '';
    position: absolute;
    bottom: 8px;
    width: 0;
    height: 0;
    border-style: solid;
    left: -25px;
    border-width: 12px 24px 12px 0;
    border-color: transparent var(--main-dark) transparent transparent;
}

.price-section .single-task .task-title,
.how-to-use .single-task .task-title {
    color: var(--pitch-dark);
    font-size: 32px;
}


.price-section .single-task .task-desc,
.how-to-use .single-task .task-desc {
    font-size: 22px;
}

/* Change bullets color. */
section.how-to-use ul.how-to-use-list li.show-single-card::after {
    background-color: var(--main-dark);
}
```

### <span id="custom-user-model">Custom User Model.</span>
In the following code, I override Django's user model to enable users to log in using their email address instead of a username.</br>
*<a href="#list-of-file"> Back to files list ^ </a>*

###### Django imports.
```python
# Django impots.
from django.db import models
from django.contrib.auth.models import AbstractUser, UserManager
from django.utils.translation import gettext_lazy as _
```

###### Overide the Django's User Manager.
```python
class EvropidanUserManager(UserManager):
    """ 
    This class inherits from UserManager to customize user creation.
    It overrides the _create_user method to require an email and normalize it, 
    sets the password, sets is_staff and is_superuser to False by default and
    sets is_staff and is_superuser to True for Super User.
    """

    # Custom method to add the email field and set the password.
    def _create_user(self, email, password, **extra_fields):
        if not email:
            raise ValueError("Email must be set")
        
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    # Create the user.
    def create_user(self, email, password=None, **extra_fields):
        extra_fields.setdefault("is_staff", False)
        extra_fields.setdefault("is_superuser", False)
        return self._create_user(email, password, **extra_fields)

    # Create the super user.
    def create_superuser(self, email, password, **extra_fields):
        extra_fields.setdefault("is_staff", True)
        extra_fields.setdefault("is_superuser", True)

        if extra_fields.get("is_staff") is not True:
            raise ValueError("Superuser must have is_staff=True.")
        if extra_fields.get("is_superuser") is not True:
            raise ValueError("Superuser must have is_superuser=True.")

        return self._create_user(email, password, **extra_fields)
    
```
###### Overide the Django's User model.
```python
class User(AbstractUser):
    """
    Custom User model.
    """

    # Override the username field by email.
    username = None
    email = models.EmailField(
        _("email address"),
        unique=True,
    )

    # Custom user manager.
    objects = EvropidanUserManager()

    # Make the email field unique.
    USERNAME_FIELD = "email"
    REQUIRED_FIELDS = []

    def __str__(self):
        return self.email
``` 

### <span id="on-scrolling">Smooth Appearance of Elements on Scrolling - JavaScript.</span>
The following JavaScript code makes the elements appear smoothly on scrolling as they enter the viewport. It takes a dictionary as a parameter. The key of this dictionary is the selector of the element, and the value is an array of the class attributes which add the desired CSS transtion effect to the element, like scrolling from the left, fading in, and so on.</br>
*<a href="#list-of-file"> Back to files list ^ </a>*

###### JavaScript code.
```javascript
class AnimateSegment {
    constructor(animeData)  {
        this.animeData = animeData;
        this.margin = 178;
    }

    // Executor.
    run = () =>  {

        // Extract keys and values from the dictionary.
        for ( const [selector, classes] of Object.entries(this.animeData)) {
            
            // Grab all elements by selector.
            const elements = document.querySelectorAll(selector);

            if (elements.length) { // If element exists.

                elements.forEach(element => {

                    // If element is in view. 
                    if (this.isInViewport(element)) {
                        // Add classes.
                        element.classList.add(...classes);
                    }
                    
                    // If element is out view. 
                    if (this.isOutViewport(element)) {
                        // Remove classes.
                        element.classList.remove(...classes);
                    }
                })
            }
        }

    }

    // Check if the element is inside of window.
    isInViewport(element){
        const rect = element.getBoundingClientRect();
        return (
            rect.top >= 0 &&
            //rect.left >= 0 &&
            rect.bottom <=
            (window.innerHeight)
        );
    };

    // Check if the element is ouside the window.
    isOutViewport(element) {
        // Get the element boundaries.
        const rect = element.getBoundingClientRect();

        // Outside of the window.
        return (
            rect.top <= -this.margin  || 
            rect.bottom >= (   
                    window.innerHeight || 
                    document.documentElement.clientHeight
                ) + this.margin
        );
    }
}
```
###### Instantiate the class.
```javascript
// Data for html elements animation.
// The key represents a selector and
// the value the css classes for transition.
const animeData = {
    '.logo': ['logo-anime', 'logo-anime1'],
    '.desktop-menu': ['desktop-menu-anime'],
    '.welcome-title': ['welcome-title-anime'],
    '.welcome-text': ['welcome-text-anime'],
    '.register-btn': ['register-btn-anime'],
    ...
}

// Segment animation.
const card = new AnimateSegment(animeData);
```

### <span id="load-shop-list">Load the Shop List - JavaScript.</span>
The following JavaScript code loads the list of the shops on page load and when the user clicks on the 'load more' button. It fetches the list of shops from the server, checks if all images of the shops are loaded, creates the HTML structure, and renders it.</br>
*<a href="#list-of-file"> Back to files list ^ </a>*
```javascript
class ShopList {
    constructor (url) {
        this.url = url; // Fetch API url.
        this.appendedID = ''; // ID for new fetched post container.
        this.rowID = ''; // Appended row id.
        this.appendedRow = ''; // Newly appended row.
        this.postWrapper = ''; // After click the newly loaded posts wrapper.  
        this.runProcess(); // Fetch new posts.
    }

    
    async runProcess() {
        this.grabElements(); // STEP-O1: Grab necessary elements.
        this.addEventToTheButton(); // STEP-O2: Add the click event to the button.
        this.data = await this.getPosts(); // STEP-O3: Fetch new posts.
        this.renderShops(); // STEP-O4: Render new posts.
        if (this.click) { // Truck post image loading.
            await this.allPostLoaded();
        }  
    }

    // Grab all necessary elements.
    grabElements() { // STEP-O1.
        this.loadMoreBtn = document.getElementById('l-more');  // Loading button.
        this.loadUpbtn = document.getElementById('l-up'); // Close loaded posts btn.
        this.loadContent = document.querySelector('.load-content'); // Loading button content.
        this.shopLoading = document.querySelector('.shop-loading'); // Loading icon.
    }

    // Add a click event on the button.
    addEventToTheButton() { // STEP-O2.
        // Load more post button.
        this.loadMoreBtn.addEventListener('click', (e) => {
            e.preventDefault();
            this.click = true;
            this.eventFunc();
        });

        // Close loaded posts button.
        this.loadUpbtn.addEventListener('click', (e) => {
            e.preventDefault();
            this.loadUpFunc();
        });
    }

    // Load more event listener function.
    async eventFunc() {
        this.index = 0;

        this.loadMoreBtn.disabled = true; // Disable button.
        this.loadMoreBtn.classList.add('disabled-btn'); // Disabled button styles.
        this.loadContent.classList.add('d-none'); // Hide button content.
        this.shopLoading.classList.add('show'); // Show loading. 

        // Grab the next page url.
        if (this.data.next) {
            this.url = this.data.next;
        }
        
        // Grab next page data.
        await this.runProcess();
    }
    
    // Close loaded posts event listener function.
    loadUpFunc() {
        // Grab the necessary elements.
        const loadedBlocks = document.querySelectorAll('.shop-appended');
        const shopHeading = document.getElementById('shop-list');

        loadedBlocks.forEach(block => {
            block.remove(); // Remove loaded posts.
            this.url = `http://127.0.0.1:8000/api/shoplist/?page=2&page_size=16`
            this.click = false; // Truck the click event.
            this.index = 0;
            this.appendedID = ''; // ID for new fetched post container.
            
            // Toggle visibilities of the read more and load up buttons.
            this.loadMoreBtn.classList.remove('d-none');
            this.loadUpbtn.classList.remove('d-block');

            shopHeading.scrollIntoView({ behavior: 'smooth' });
        });
    }

    // Check if all posts images has been loaded.
    async allPostLoaded() {
        const images = document.querySelectorAll(`.shop-list #${this.appendedID} img`);
        var totalImages = images.length;
        let loadedImages = 0;

        images.forEach(img => {
            if (img.complete) {
                loadedImages++;
            } else {
                img.onload = () => {
                    loadedImages++;
                    if (loadedImages === totalImages) {
                        this.allImagesLoaded();
                    }
                };
            }
        });

        if (loadedImages === totalImages ) {
            this.allImagesLoaded();
        }
    }

    // When all images has been loaded.
    allImagesLoaded() {
        // Grab the new posts container.
        const appended = document.querySelector(`#${this.appendedID}`);
        // Grab new posts rows.    
        const newRows = document.querySelectorAll(`#${this.appendedID} .shop-row`);

        appended.classList.remove('d-none'); // Show container.
        appended.classList.add('shop-fade-in'); // Add CSS animation to the container.

        this.loadMoreBtn.disabled = false; // Disable button.
        this.loadMoreBtn.classList.remove('disabled-btn'); // Disabled button styles.

        this.loadContent.classList.remove('d-none'); // Hide button content.
        this.shopLoading.classList.remove('show'); // Hide loading.

        // Toggle the show more and load up buttons if there are no more posts.
        if(!this.data.next) {
            this.hideButton(); 
        }
        
        // As soon as the css animation finishes,
        // add the trans and fade-in classes to the new shop-row.
        appended.addEventListener('animationend', () => {
            newRows.forEach(row => {
                row.classList.add('trans', 'fade-in');
            });
        });

        appended.scrollIntoView({ 
            behavior: 'smooth',
            block: 'start' 
        });
        
    }

    // If there is no more post on the next page, hide the button.
    hideButton() {
        this.loadMoreBtn.classList.add('d-none');
        this.loadUpbtn.classList.add('d-block');
    }

    // Try to fetch new posts otherwise rise an error.
    async getPosts() { // STEP-O3.
        try {
            const data = await this.loadData();
            return data;
        } catch (error) {
            console.log('Error', error);
        }
    }

    // Fetch new post, convert to JSON and return it.
    async loadData() {
        const response = await fetch(`${this.url}`);
        const data = await response.json();
        return data;
    }

    // Create HTML and render it.
    renderShops() { // STEP-O4.
        // Grab the posts container.
        var container = document.querySelector('.shop-list');

        // After click, wrappe the newly loaded posts with wrapper.
        if (this.click) {
            container.appendChild(this.appendedWrapper());
        }

        // Loop through new posts.
        this.data.results.forEach((shop, index) => {
            
            // Append new row.
            if (index % this.postIndex === 0) {
                if(this.click){
                    this.postWrapper.appendChild(this.row());
                } else {
                    container.appendChild(this.row());
                }
            }

            // Crete col and append into row.
            const col = this.col(shop);
            this.appendedRow.appendChild(col);

        });
    }


    // The wrapper for the newly loaded posts after clicking on the button.
    appendedWrapper() {
        const elm = document.createElement('div');

        elm.classList.add('shop-appended', 'd-none');
        this.appendedID = `appended-${this.appendedID}`;
        elm.id = this.appendedID;

        this.postWrapper = elm;
        return elm;
    }

    // Get index for posts per row based on the devices.
    get postIndex() {
        const wIw =  window.innerWidth;
        if (wIw <=576) {
            return 2;
        } else if (wIw > 576 && wIw <=992) {
            return 3;
        } else {
            return 4;
        }
    }

    // Create the row element.
    row() {
        const row = document.createElement('div');
        this.rowID = `row-${this.generateUniqueId}`;
        if (!this.click){
            row.classList.add('row', `row-cols-${this.postIndex}`, 'align-items-center', 'shop-row', 'text-center', 'trans');
        } else {
            row.classList.add('row', `row-cols-${this.postIndex}`, 'align-items-center', 'shop-row', 'text-center');
        }

        row.id = this.rowID;

        this.appendedRow = row;
        return row;
    }
  
    // Generate unique ID.
    get generateUniqueId() {
        return Math.random().toString(16).slice(2, 10);
    }

    // Create post element.
    col(shop) {
        // Create the necessary elements.
        const col = document.createElement('div');
        const link = document.createElement('a');
        const img = document.createElement('img');

        // Add attributes.
        col.classList.add('col', 'text-center');
        link.classList.add('shop-link', 'border');
        link.href = `${shop.link}`;
        link.target = `_blank`;
        img.src = `${shop.image}`;

        link.appendChild(img); // Append the image into the link.
        col.appendChild(link); // Append the link into the column.

        return col;
    }

}
```

### <span id="send-email">Send the Email.</span>
The following code sends an email to the user upon button click and also when the user confirms his/her email address. It uses Django signals to listen for email confirmation and triggers the appropriate code.</br>
*<a href="#list-of-file"> Back to files list ^ </a>*

##### <span id="send-email-section"> Send Email Section Links: </span>
-   [The View of the Send Email](#send-email-view)
-   [signals.py](#email-signals)
-   [Send Email JavaScript code](#send-email-js-code)

###### <span id="send-email-view">The View of the Send Email.</span>
*<a href="#send-email"> Back to links ^ </a>*
```python
class SendEmail(LoginRequiredMixin, View):
    """
    This is the send email view. Which is used by Fetch API.
    It sends an email at the email confirmation event and whenevere user
    clicks on the resend button.
    """
    def post(self, request, *args, **kwargs ):
        user_email = request.user.email
        self.send_email_to_user(user_email)
        return JsonResponse({'status': 'Email sent'})
    
    def send_email_to_user(self, user_email):
        subject = 'ევროპული მისამართის მონაცემები.' 
        html_message = render_to_string('user_profile/email_text.html')
        text_message = strip_tags(html_message) # Remove HTML tags for fallback.
        
        # Prepare the email message.
        message = EmailMultiAlternatives(
            subject=subject,
            body=text_message,
            from_email='evropidan.ge',
            to=[user_email],
        )

        # Attach the HTML message.
        message.attach_alternative(html_message, 'text/html')
        
        # Send the email.
        message.send()
```
###### <span id="email-signals">signals.py</span>
*<a href="#send-email"> Back to links ^ </a>*
```python
# Django imports.
from django.dispatch import receiver
from django.contrib.auth import get_user_model

# Third-party imports.
from allauth.account.signals import email_confirmed 

# Local imports.
from user_profile.models import UserProfile
from user_profile.views import SendEmail


@receiver(email_confirmed)
def send_email_on_confirm(request, **kwargs):

    # Grab the email from the kwargs and then the user by that email.
    confirmed_email = kwargs['email_address']
    user = get_user_model().objects.get(email=confirmed_email)

    # Create the record if it isnot there already.
    record, created = UserProfile.objects.get_or_create(
        user=user,
        email_sent=True,
    )

    # Send email.
    view = SendEmail()
    view.send_email_to_user(kwargs['email_address'])

    # Update the record.
    if not created:      
        record.email_sent = True         
        record.save()
```
###### <span id="send-email-js-code">The JavaScript Code of the Send Email.</span>
*<a href="#send-email"> Back to links ^ </a>*
```javascript
// This class handles the CSRF token cookie.
class Cookie {
    constructor(name) {
        this.name = name;
    }

    // This function decodes the cookie value.
    decodeCookie(cookie) {
            return decodeURIComponent(cookie.split('=')[1]);
    }

    // This function gets the cookie value.
    get(){
        const cookieValue = document.cookie
        .split('; ')
        .find(raw => raw.startsWith(this.name + '='));
        return cookieValue ? this.decodeCookie(cookieValue) : null;

    }
}


/* This class adds event listeners to the send email buttons,
    sends the email to the server and toggles the spinner icon appearance. */
class SendEmail {
    constructor(url) {
        this.url = url;
        this.csrfToken = new Cookie('csrftoken');
        this.buttons = document.querySelectorAll('.send-email');
        this.spinnerElm = document.querySelector('.profile-wrapper .spinner-border');
        this.addEvents();
    }

    /* Add event listeners to the send email buttons. */
    addEvents() {               
        this.buttons.forEach(button => {
            button.addEventListener('click', (e) => {
                e.preventDefault();

                const button = e.target // Button that was clicked.
                const spinner = e.target.parentNode.querySelector('.spinner-border'); // Current spinner icon.
                
                this.showSpinner(button, spinner); // Show spinner icon.
                this.sendEmail(button, spinner); // Send email to the server.
            });
        });
    }

    // Show spinner icon.
    showSpinner(button, spinner) {
        button.classList.add('d-none'); //  Hide the button.
        spinner.classList.remove('d-none'); // Show the spinner.
    }

    // Send email to the server.
    sendEmail(button, spinner) {
        fetch(this.url, {
            method: 'POST',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json',
                'X-CSRFToken': this.csrfToken.get()
            }
        }).then(response => {
            if (response.status === 200) {
                this.hideSpinner(button, spinner); // Hide spinner.
            } else {
                console.log('ვერ მოხერხდა გაგზავნა.');
            }
        });
    }

    // Hide spinner icon.
    hideSpinner(button, spinner) {
        button.classList.remove('d-none'); // Show the button.
        spinner.classList.add('d-none'); // Hide the spinner.
    }

}

// Initialize the class.
new SendEmail('/profile/send-email/');
```
