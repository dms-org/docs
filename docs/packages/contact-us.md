Contact Us Package
==================

Most website's include a contact form.
The contact us packages allows these enquires to be stored in the backend.

## Installation

### Install the package via composer

`composer require dms-org/package.contact-us`

### Register the package in `app/AppCms.php`

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Core\Cms;
use Dms\Core\CmsDefinition;
use Dms\Package\ContactUs\Cms\ContactUsPackage;

/**
 * The application's cms.
 *
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class AppCms extends Cms
{
    /**
     * Defines the structure and installed packages of the cms.
     *
     * @param CmsDefinition $cms
     *
     * @return void
     */
    protected function define(CmsDefinition $cms)
    {
        $cms->packages([
            // Add this line to register the package...
            'contact-us' => ContactUsPackage::class,
        ]);
    }
}
```

### Register the ORM in `app/AppOrm.php`

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Core\Persistence\Db\Mapping\Definition\Orm\OrmDefinition;
use Dms\Core\Persistence\Db\Mapping\Orm;
use Dms\Package\ContactUs\Persistence\ContactUsOrm;

/**
 * The application's orm.
 *
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class AppOrm extends Orm
{
    /**
     * Defines the object mappers registered in the orm.
     *
     * @param OrmDefinition $orm
     *
     * @return void
     */
    protected function define(OrmDefinition $orm)
    {
        // Add this line to register the mappers
        $orm->encompass(new ContactUsOrm());
    }
}
```

## Usage

### Sample contact form

```html
<form action="..." method="post">
   {!! csrf_field() !!}
   
   <div class="form-group">
       <label>Your Name</label>
       <input type="text" class="form-control" required name="name">
   </div>
   <div class="form-group">
       <label>Email</label>
       <input type="email" class="form-control" required name="email">
   </div>
   <div class="form-group">
       <label>Subject</label>
       <input type="text" class="form-control" required name="subject">
   </div>
   <div class="form-group">
       <label>Your Message</label>
       <textarea name="message" class="form-control" required></textarea>
   </div>

   <button type="submit" class="btn btn-primary">Submit Enquiry</button>
</form>
```

### Record contact enquiry and send notification

```php
<?php declare(strict_types = 1);

namespace App\Http\Controllers;

use Dms\Package\ContactUs\Core\ContactEnquiry;
use Dms\Package\ContactUs\Core\ContactEnquiryService;
use Illuminate\Http\Request;
use Illuminate\Mail\Message;

/**
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class ContactController extends Controller
{
    public function submitEnquiry(Request $request, ContactEnquiryService $contactEnquiryService)
    {
        $this->validate($request, [
            'name'    => 'required',
            'email'   => 'required|email',
            'subject' => 'required',
            'message' => 'required',
        ]);

        $contactEnquiryService->recordEnquiry(
            $request->input('email'),
            $request->input('name'),
            $request->input('subject'),
            $request->input('message'),
            function (ContactEnquiry $enquiry) {
                // Send the notification email...
            }
        );

        return redirect('contact/success');
    }
}
```