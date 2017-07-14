Analytics Package
=================

The analytics package integrates with Google Analytics to provide basic analytics
on the dashboard.

![Dashboard](/resources/images/cms/dashboard-1.jpg)

## Installation

This package comes pre-installed with the DMS.

## Usage

After configuring the Google Analytics settings in the backend you should ensure the analytics scripts
are output to all your pages on the site.

Assuming you have a template blade view you can use this package to output the scripts: 

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
	<head>
	    ...
	    
	    <!-- Output analytics scripts at the end of document head  -->
		{!! app(\Dms\Package\Analytics\AnalyticsEmbedCodeService::class)->getEmbedCode() !!}
	</head>

    <body>
        ...
    </body>
</html>
```