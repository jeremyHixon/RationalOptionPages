# RationalOptionPages

A PHP class for building WordPress Option Pages. Uses multi-dimensional associative arrays to try and make the process of adding option pages a little easier to map out and use.

## Table of Contents

1. [Installation](#installation)
2. [Usage](#usage)
	1. [Pages](#pages)
	2. [Subpages](#subpages)
	3. [Fields](#fields)
3. [Retrieving Data](#retrieving-data)
4. [To Do](#to-do)

## Installation

* Download or clone the repo
* Include `RationalOptionPages.php` in your file
* Instantiate the class with your array of pages

```php
if ( !class_exists( 'RationalOptionPages' ) ) {
	require_once('RationalOptionPages.php');
}
$pages = array(
	'sample-page'	=> array(
		'page_title'	=> __( 'Sample Page', 'sample-domain' ),
	),
);
$option_page = new RationalOptionPages( $pages );
```
### Note:

If you installed this previously and the class is already being instantiated somewhere else you will get fatal errors. You can avoid this by either renaming the class or by wrapping the `require_once()` in a `if ( !class_exists() )` conditional.

## Usage

### Pages

Pages are added in a key/value syntax where the value is an array of parameters for the page itself.

#### Parameters

Based on [WordPress' `add_menu_page()` function](https://developer.wordpress.org/reference/functions/add_menu_page/).

* `page_title` - The text string for the title of the page __Required__.
* `menu_title` - The text string for the menu item itself. Defaults to `page_title`.
* `capability` - The permissions required to access this page. Defaults to `manage_options`.
* `menu_slug` - The "slug" of the menu item. Defaults to a "slugified" version of the `page_title`.
* `icon_url` - The URL of the menu icon. [WordPress' Dashicons are available](https://developer.wordpress.org/resource/dashicons). Defaults to `false`, renders "dashicons-admin-generic".
* `position` - The position where the menu item appears, from 1 to 99. Defaults to `null`, last.

If your page title is more than 2 words I recommend using the `menu_title` parameter to avoid wrapping. The default `capability` should work for most plugins and themes as only the admin would typically make higher level changes like these.

### Subpages

Subpages are differentiated by the `parent_slug` parameter. They can be added either at the top level of the pages array submitted to the class or under a `subpages` key within another, top-level page parameter array.

#### Parameters

Based on [WordPress' `add_submenu_page()` function](https://developer.wordpress.org/reference/functions/add_submenu_page/).

* `parent_slug` - The `menu_slug` of the parent page. [WordPress has several top-level menu items](https://codex.wordpress.org/Administration_Menus#Using_add_submenu_page). __Required__ (unless added via the `subpages` key method).
* `page_title` - The text string for the title of the page. __Required__.
* `menu_title` - The text string for the menu item itself. Defaults to `page_title`.
* `capability` - The permissions required to access this page. Defaults to `manage_options`.
* `menu_slug` - The "slug" of the menu item. Defaults to a "slugified" version of the `page_title`.

#### Example

```php
require_once('RationalOptionPages.php');
$pages = array(
	'sample-page'	=> array(
		'page_title'	=> __( 'Sample Page', 'sample-domain' ),
		// via the subpages key
		'subpages'		=> array(
			'sub-page-one'	=> array(
				'page_title'	=> __( 'Sub Page One', 'sample-domain' ),
			),
		),
	),
	// via the pages array itself
	'sub-page-two'	=> array(
		'parent_slug'	=> 'sample_page',
		'page_title'	=> __( 'Sub Page Two', 'sample-domain' ),
	),
	// sub page of the "Appearance" menu item
	'sub-theme'	=> array(
		'parent_slug'	=> 'themes.php',
		'page_title'	=> __( 'Sub Theme', 'sample-domain' ),
	),
);
$option_page = new RationalOptionPages( $pages );
```

### Sections

Sections are added via a `sections` key in the page parameter arrays.

#### Parameters

Based on [WordPress' `add_settings_section()` function](https://developer.wordpress.org/reference/functions/add_settings_section/).

* `title` - The title of the section. __Required__.
* `id` - The ID of the section.
* `callback` - An optional parameter for generating custom, section content. __Requires `custom` parameter be set to `true`__.
* `custom` - A boolean option that indicates you want to use a custom callback. Defaults to `false`.
* `text` - An option parameter for adding HTML text under the section title.
* `include` - An option parameter that calls PHP's `include()` under the section title. __Use absolute path__.

#### Example

```php
require_once('RationalOptionPages.php');
$pages = array(
	'sample-page'	=> array(
		'page_title'	=> __( 'Sample Page', 'sample-domain' ),
		'sections'		=> array(
			'section-one'	=> array(
				'title'			=> __( 'Section One', 'sample-domain' ),
				'text'			=> '<p>' . __( 'Some HTML text to describe the section', 'sample-domain' ) . '</p>',
				'include'		=> plugin_dir_path( __FILE__ ) . '/your-include.php',
			),
			'section-two'	=> array(
				'title'			=> __( 'Section Two', 'sample-domain' ),
				'custom'		=> true,
				'callback'		=> 'custom_section_callback_function',
			),
		),
	),
);
$option_page = new RationalOptionPages( $pages );
```

### Fields

Fields are added via a `fields` key in the section parameter array.

#### Parameters

Based on [WordPress' `add_settings_field()` function](https://developer.wordpress.org/reference/functions/add_settings_field/).

* `title` - The title/label of the field. __Required__.
* `id` - The ID of the field.
* `callback` - An optional parameter for generating custom, field content. __Requires `custom` parameter be set to `true`__.
* `custom` - A boolean option that indicates you want to use a custom callback. Defaults to `false`.
* `type` - The type of field to use. [Most input types are available](http://www.w3schools.com/html/html_form_input_types.asp) as well as `select`, `textarea`, `wp_editor` and `media` (instead of `file` input type).
* `text` - Help text for most input types. Label for `checkbox`.
* `title_attr` - The "title" attribute of the input (if available).
* `choices` - Associative array of options for radio groups and select elements. __Required for `radio` and `select` types__.
* `placeholder` - The placeholder attribute of the input (if available).
* `value` - The default value of the input.
* `checked` - A boolean for the default state of the `checkbox` type. Defaults to `false`.
* `attributes` - Associative array of additional attributes for the input element.
	* Input - `autocomplete, autofocus, disabled, list, max, maxlength, min, pattern, readonly, required, size and step`
	* Select - `multiple, size`
	* Textarea - `cols, rows and wrap`
* `sanitize` - A boolean that indicates whether or not the field's value should be sanitized. Defaults to `false`, and doesn't apply to checkboxes.
	
#### Examples

The most basic of inputs.

```php
require_once('RationalOptionPages.php');
$pages = array(
	'sample-page'	=> array(
		'page_title'	=> __( 'Sample Page', 'sample-domain' ),
		'sections'		=> array(
			'section-one'	=> array(
				'title'			=> __( 'Section One', 'sample-domain' ),
				'fields'		=> array(
					'default'		=> array(
						'title'			=> __( 'Default', 'sample-domain' ),
					),
				),
			),
		),
	),
);
$option_page = new RationalOptionPages( $pages );
```

Almost everything.

```php
require_once('RationalOptionPages.php');
$pages = array(
	'sample-page'	=> array(
		'page_title'	=> __( 'Sample Page', 'sample-domain' ),
		'sections'		=> array(
			'section-one'	=> array(
				'title'			=> __( 'Standard Inputs', 'sample-domain' ),
				'fields'		=> array(
					'default'		=> array(
						'title'			=> __( 'Default (text)', 'sample-domain' ),
						'text'			=> __( 'Text attributes are used as help text for most input types.' ),
					),
					'date'			=> array(
						'title'			=> __( 'Date', 'sample-domain' ),
						'type'			=> 'date',
						'value'			=> 'now',
					),
					'datetime'		=> array(
						'title'			=> __( 'Datetime-Local', 'sample-domain' ),
						'type'			=> 'datetime-local',
						'value'			=> 'now',
					),
					'datetime-local' => array(
						'title'			=> __( 'Datetime-Local', 'sample-domain' ),
						'type'			=> 'datetime-local',
						'value'			=> 'now',
					),
					'email'			=> array(
						'title'			=> __( 'Email', 'sample-domain' ),
						'type'			=> 'email',
						'placeholder'	=> 'email.address@domain.com',
					),
					'month'			=> array(
						'title'			=> __( 'Month', 'sample-domain' ),
						'type'			=> 'month',
						'value'			=> 'now',
					),
					'number'		=> array(
						'title'			=> __( 'Number', 'sample-domain' ),
						'type'			=> 'number',
						'value'			=> 42,
					),
					'password'		=> array(
						'title'			=> __( 'Password', 'sample-domain' ),
						'type'			=> 'password',
					),
					'search'		=> array(
						'title'			=> __( 'Search', 'sample-domain' ),
						'type'			=> 'search',
						'placeholder'	=> __( 'Keywords or terms&hellip;', 'sample-domain' ),
					),
					'tel'			=> array(
						'title'			=> __( 'Telephone', 'sample-domain' ),
						'type'			=> 'tel',
						'placeholder'	=> '(555) 555-5555',
					),
					'time'			=> array(
						'title'			=> __( 'Time', 'sample-domain' ),
						'type'			=> 'time',
						'value'			=> 'now',
					),
					'url'			=> array(
						'title'			=> __( 'URL', 'sample-domain' ),
						'type'			=> 'url',
						'placeholder'	=> 'http://jeremyhixon.com',
					),
					'week'			=> array(
						'title'			=> __( 'Week', 'sample-domain' ),
						'type'			=> 'week',
						'value'			=> 'now',
					),
				),
			),
			'section-two'	=> array(
				'title'			=> __( 'Non-standard Input', 'sample-domain' ),
				'fields'		=> array(
					'checkbox'		=> array(
						'title'			=> __( 'Checkbox', 'sample-domain' ),
						'type'			=> 'checkbox',
						'text'			=> __( 'Text attributes are used as labels for checkboxes' ),
					),
					'color'			=> array(
						'title'			=> __( 'Color', 'sample-domain' ),
						'type'			=> 'color',
						'value'			=> '#cc0000',
					),
					'media'			=> array(
						'title'			=> __( 'Media', 'sample-domain' ),
						'type'			=> 'media',
						'value'			=> 'http://your-domain.com/wp-content/uploads/2016/01/sample.jpg',
					),
					'radio'			=> array(
						'title'			=> __( 'Radio', 'sample-domain' ),
						'type'			=> 'radio',
						'value'			=> 'option-two',
						'choices'		=> array(
							'option-one'	=> __( 'Option One', 'sample-domain' ),
							'option-two'	=> __( 'Option Two', 'sample-domain' ),
						),
					),
					'range'			=> array(
						'title'			=> __( 'Range', 'sample-domain' ),
						'type'			=> 'range',
						'value'			=> 75,
					),
					'select'		=> array(
						'title'			=> __( 'Select', 'sample-domain' ),
						'type'			=> 'select',
						'value'			=> 'option-two',
						'choices'		=> array(
							'option-one'	=> __( 'Option One', 'sample-domain' ),
							'option-two'	=> __( 'Option Two', 'sample-domain' ),
						),
					),
					'select-multiple'		=> array(
						'title'			=> __( 'Select multiple', 'sample-domain' ),
						'type'			=> 'select',
						'value' => array(
							'option-two'
						),
						'choices' => array(
							'option-one' => __( 'Option One', 'sample-domain' ),
							'option-two' => __( 'Option Two', 'sample-domain' ),
							'option-three' => __( 'Option Three', 'sample-domain' ),
						),
						'attributes' => array(
							'multiple' => 'multiple'
						),
						'sanitize' => true
					),
					'textarea'		=> array(
						'title'			=> __( 'Textarea', 'sample-domain' ),
						'type'			=> 'textarea',
						'value'			=> 'Pellentesque consectetur volutpat lectus, ac molestie lorem molestie nec. Vestibulum in auctor massa. Vivamus convallis nunc quis lacus maximus, non ultricies risus gravida. Praesent ac diam imperdiet, volutpat nisi sed, semper eros. In nec orci hendrerit, laoreet nunc eu, semper magna. Curabitur eu lorem a enim sodales consequat. Vestibulum eros nunc, congue sed blandit in, maximus eu tellus.',
					),
					'wp_editor'		=> array(
						'title'			=> __( 'WP Editor', 'sample-domain' ),
						'type'			=> 'wp_editor',
						'value'			=> 'Pellentesque consectetur volutpat lectus, ac molestie lorem molestie nec. Vestibulum in auctor massa. Vivamus convallis nunc quis lacus maximus, non ultricies risus gravida. Praesent ac diam imperdiet, volutpat nisi sed, semper eros. In nec orci hendrerit, laoreet nunc eu, semper magna. Curabitur eu lorem a enim sodales consequat. Vestibulum eros nunc, congue sed blandit in, maximus eu tellus.',
					),
				),
			),
		),
	),
);
$option_page = new RationalOptionPages( $pages );
```

## Retrieving Data

Each page stores it's fields in an entry in the database. The key is the array key for your page.

Using the example above as a reference:

```php
// Get all options for the page
$options = get_option( 'sample-page', array() );

// Each field id is a key in the options array
$date = $options['date'];
$tel = $options['telephone'];
```

If you let the class generate the field IDs then they will be "slugified" versions of the `title` parameter. For example; a field with the title of "Website Address" will have an ID of "website_address". You can also see this key by inspecting the input and looking at the input's `name` attribute. Within the square (`[]`) brackets.

## To Do

* Add `text` and `include` parameters to pages.
