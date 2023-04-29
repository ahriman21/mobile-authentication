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
```

AUTHENTICATION_BACKENDS = [
  'django.contrib.auth.backends.ModelBackend', # the default one.
  'your_app.project_backend.MobileBackend',    # my custom backend.
]

```

### 3- in your custom user model you must put your project backend (MobileBackend) in a variable named backend .
```
class User(AbstractBaseUser):
  phone = models.CharField(max_length=11,unique=True)
  otp_code = models.CharField(max_length=6)
  otp_created_time = models.DateTime(auto_now=True)
  ...
  backend = 'your_app.project_backend.MobileBackend'
```

### 4- 
