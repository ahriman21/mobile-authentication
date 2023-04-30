# mobile-authentication
This is a guide about authenticating user via phone number only.
it means user can register and login in your webapp with a phone number and a one time code for verifying user.

### 1- creating a custom model backend for authentication :
create a file and call it project_backend.py
```
from django.contrib.auth.backends import ModelBackend
from your_user_app.models User

class MobileBackend(ModelBackend):
  def authenticate(request, username=None, password=None, **kwargs):
    phone = kwargs['phone']
    try:
      user = User.objects.get(phone=phone)
    except User.DoesNotExist:
      pass

```

### 2- edit model backend in django settings :
settings.py
```

AUTHENTICATION_BACKENDS = [
  'django.contrib.auth.backends.ModelBackend', # the default one.
  'your_app.project_backend.MobileBackend',    # my custom backend.
]

```

### 3- in your custom user model you must put your project backend (MobileBackend) in a variable named backend .
models.py
```
class User(AbstractBaseUser):
  phone = models.CharField(max_length=11,unique=True)
  otp_code = models.CharField(max_length=6)
  otp_created_time = models.DateTime(auto_now=True)
  ...
  backend = 'your_app.project_backend.MobileBackend'
```

### 4- create and send otp code on user's phone :
util.py
```
from random import randint

def create_otp():
  return randint(100000,999999)
 
 
def send_otp(phone, otp):
  pass
  
 # use the functions in register view.
 
def check_otp_expiration(user):
   """ if this function return False it means otp is expired."""
  now = datetime.now()
  otp_create_time = user.otp_created_time
  diff_time = now - otp_create_time
  if diff_time.seconds > 120 :
    return False
    
  return True
```

### 5- verify user's phone :
views.py
```
def verify_view(request):
  user_phone = request.session.get('user_phone')
  user = User.objects.get(phone=user_phone)
  form = VerifyUserPhoneForm()
  if request.method == 'POST':
    if user.otp == form.cleaned_data.get('otp'):
      user.is_active = True
      user.save()
      login(request,user)
      redirect('home')
  context = {'form' = form}
  return render(request,template_name,context)

```
