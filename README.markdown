# Form Controls
An XSLT utility to create powerful HTML forms with [Symphony](http://symphony-cms.com)

## What is it?
Form Controls is a suite of XSL templates for rapidly building forms that are tightly coupled with Symphony Events. The core aim is to make forms easier to build so that the developer can spend less time on validation and checking posted data, and more time on adding the extra layer of polish that make forms more usable.

Form Controls provides the following functionality:

* templates to render all common HTML form control elements
* pre-populate controls with static or dynamic data
* persist posted values from Events
* associate and optionally wrap with labels
* provides HTML hooks when field is invalid (class attribute)
* provides powerful validation messages and error response

## Download
Form Controls (form-controls.xsl) is a single XSLT file and can be downloaded from Github:
<http://github.com/nickdunn/form-controls/tree/master>

## Installing the utility
Add the `form-controls.xsl` file to your `/workspace/utilities` folder alongside the [other cool XSLT utilities](http://symphony21.com/downloads/xslt/) you know and love.

On the page in which you are building the form, import the XSL file:

	<xsl:import href="../utilities/form-controls.xsl"/>

Form Controls uses some functions not available to XSLT 1.0, therefore you will need to have the [EXSLT library](http://exslt.org/) installed (you should do already) and you will need to add the EXSLT namespace to your page. Additionally Form Controls adds all templates and variables to `form` namespace so as not to clash with other templates in your site. After including these two namespaces, your page `stylesheet` element should look something like this:

	<xsl:stylesheet	version="1.0"
		xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
		xmlns:exsl="http://exslt.org/common"
		xmlns:form="http://nick-dunn.co.uk/xslt/form-controls"
		extension-element-prefixes="exsl form">

The `extension-element-prefixes` attribute prevents these namespaces being added to your HTML elements when the page is rendered.

## Supported form controls
Form Controls supports all field types native to Symphony (input, textarea, select) and adds a few of its own too. It can generate the following:

* Label
* Input (text, password, file)
* Textarea
* Checkbox
* Radio
* Select (including multi-select)
* Radiobutton List
* Checkbox List
* Validation Summary (list of error messages)

Form Controls assumes you are submitting to a front-end event in Symphony.

## Most basic example
Submits to an event (`save-post`) derived from a Posts section. Create a new page and attach the `Save Post` event to it. Paste the following XSLT into your page replacing the default contents:

	<?xml version="1.0" encoding="UTF-8"?>
	<xsl:stylesheet	version="1.0"
		xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
		xmlns:exsl="http://exslt.org/common"
		xmlns:form="http://nick-dunn.co.uk/xslt/form-controls"
		extension-element-prefixes="exsl form">

	<!-- Import form-controls.xsl -->
	<xsl:import href="../utilities/form-controls.xsl"/>

	<!-- Define a global variable pointing to your Event -->
	<xsl:variable name="form:event" select="/data/events/save-blog-post"/>

	<xsl:template match="data">
	
		<form action="" method="post">
		
			<fieldset>
				<legend>Create new post</legend>

				<xsl:call-template name="form:validation-summary"/>

				<label>
					Post title<br/>
					<xsl:call-template name="form:input">
						<xsl:with-param name="handle" select="'title'"/>
					</xsl:call-template>
				</label>

				<label>
					Post contents<br/>
					<xsl:call-template name="form:textarea">
						<xsl:with-param name="handle" select="'content'"/>
					</xsl:call-template>
				</label>

			</fieldset>
		
		</form>
	
	</xsl:template>
	
	</xsl:stylesheet>

This will generate HTML akin to the following (just the `fieldset` included, my indenting):

	<fieldset>
		<legend>Create new post</legend>
		<label>
			Post title<br />
			<input name="fields[title]" id="fields-title" title="" class="" type="text" value="" />
		</label>
		<label>
			Post contents<br />
			<textarea name="fields[content]" id="fields-content" title="" class=""></textarea>
		</label>
	</fieldset>
	
On form submit you will also see a validation summary — either a success message, or a list of missing or invalid fields.

## Template examples
Here are examples outlining the full range of control templates.

### form:event
This variable should be created globally, outside of the page templates. It should select the event node created by the event you are posting to.

	<xsl:variable name="form:event" select="/data/events/save-blog-post"/>

### form:label
Renders an HTML `label` element that can be explicitly assigned to another form element. Can be wrapped around other controls.

#### Parameters

* `for` (optional, string): Handle of a Symphony field name that this label is associated with
* `text` (optional, string): Text value of the label. Defaults to field name ($for value)
* `child` (optional, XML): Places this XML inside the label, for wrapping elements with the label
* `child-position` (optional, string): Place the child before or after the label text. Defaults to "after"
* `class` (optional, string): Value of the HTML @class attribute
* `section` (optional, string): Use with EventEx to change "fields[...]" to a section handle
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node

#### Example

	<xsl:call-template name="form:label">
		<xsl:with-param name="for" select="'title'"/>
	</xsl:call-template>


	<xsl:call-template name="form:label">
		<xsl:with-param name="for" select="'title'"/>
		<xsl:with-param name="text" select="'Post Title'"/>
		<xsl:with-param name="child">
			<xsl:call-template name="form:input">
				<xsl:with-param name="handle" select="'title'"/>
			</xsl:call-template>
		</xsl:with-param>
	</xsl:call-template>

### form:input
Renders an HTML text `input` element with support for `password` and `file` types.

#### Parameters

* `handle` (mandatory, string): Handle of the field name
* `value` (optional, string): Initial value of form control. Will not work for `file` inputs.
* `type` (optional, string): Type attribute value ("text", "password" "file"). Defaults to "text"
* `class` (optional, string): Class attribute value
* `title` (optional, string): Title attribute value
* `size` (optional, string): Size attribute value
* `maxlength` (optional, string): Maxlength attribute value
* `autocomplete` (optional, string): Autocomplete attribute value ("off"). Not set by default
* `section` (optional, string): Use with EventEx to change "fields[...]" to a section handle
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node
	
#### Example

	<xsl:call-template name="form:input">
		<xsl:with-param name="handle" select="'title'"/>
		<xsl:with-param name="value" select="'My first blog post'"/>
	</xsl:call-template>

	<xsl:call-template name="form:input">
		<xsl:with-param name="handle" select="'image'"/>
		<xsl:with-param name="type" select="'file'"/>
		<xsl:with-param name="title" select="'Please upload an image'"/>
	</xsl:call-template>

### form:textarea
Renders an HTML `textarea` element. 

#### Parameters

* `handle` (mandatory, string): Handle of the field name
* `value` (optional, string): Contents of the textarea
* `class` (optional, string): Class attribute value
* `rows` (optional, string): Rows attribute value
* `cols` (optional, string): cols attribute value
* `section` (optional, string): Use with EventEx to change "fields[...]" to a section handle
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node

#### Example

	<xsl:call-template name="form:textarea">
		<xsl:with-param name="handle" select="'content'"/>
		<xsl:with-param name="rows" select="'5'"/>
		<xsl:with-param name="cols" select="'40'"/>
	</xsl:call-template>

### form:checkbox
Renders an HTML checkbox `input` element. If a checkbox is not checked, its value is never sent in the POST array to the event. For this reason a hidden field with the same name, and a value of "no" will be rendered just before the checkbox. Ticking the checkbox will override this "no" value.

#### Parameters

* `handle` (mandatory, string): Handle of the field name
* `checked` (optional, string): Initial checked state ("yes", "no"). Defaults to "no"
* `checked-by-default` (optional, string): When there is no initial $checked value (a fresh form), check by default ("yes", "no"). Defaults to "no"
* `class` (optional, string): Class attribute value
* `title` (optional, string): Title attribute value
* `section` (optional, string): Use with EventEx to change "fields[...]" to a section handle
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node
* `allow-multiple` (optional, string): Internal use only ("yes", "no"). Whether checkbox is part of a checkbox list. Defaults to "no"
* `allow-multiple-value` (optional, string): Internal use only. Overrides default "yes" value when part of a checkbox list

#### Example
	
	<!-- renders a checkbox (ticked), inside a label with the label text following the checkbox -->
	<xsl:call-template name="form:label">
		<xsl:with-param name="for" select="'published'"/>
		<xsl:with-param name="text" select="'Publish this post'"/>
		<xsl:with-param name="child">
			<xsl:call-template name="form:checkbox">
				<xsl:with-param name="handle" select="'published'"/>
				<xsl:with-param name="checked-by-default" select="'yes'"/>
			</xsl:call-template>
		</xsl:with-param>
		<xsl:with-param name="child-position" select="'before'"/>
	</xsl:call-template>

### form:radio
Renders an HTML radio `input` element. Could be used to save values to an Input or Select Box field.

#### Parameters

* `handle` (mandatory, string): Handle of the field name
* `value` (optional, string): The selected value for this radio sent when the form is submitted
* `existing-value` (optional, string): An initial value. Selects radio if it matches $value
* `checked-by-default` (optional, string): When there is no initial $existing-value (a fresh form), select by default ("yes", "no"). Defaults to "no"
* `class` (optional, string): Class attribute value
* `title` (optional, string): Title attribute value
* `type` (optional, string): Internal use only ("radio", "checkbox"). Defaults to "radio"
* `section` (optional, string): Use with EventEx to change "fields[...]" to a section handle
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node
* `allow-multiple` (optional, string): Internal use only ("yes", "no"). Whether control is part of a radio/checkbox list. Defaults to "no"

#### Example

	<!-- allow selection from two options -->
	<xsl:call-template name="form:label">
		<xsl:with-param name="text" select="'Choose Option 1?'"/>
		<xsl:with-param name="child">
			<xsl:call-template name="form:radio">
				<xsl:with-param name="handle" select="'option'"/>
				<xsl:with-param name="value" select="'Option 1'"/>
				<xsl:with-param name="checked-by-default" select="'yes'"/>
			</xsl:call-template>
		</xsl:with-param>
		<xsl:with-param name="child-position" select="'before'"/>
	</xsl:call-template>
	
	<xsl:call-template name="form:label">
		<xsl:with-param name="text" select="'Choose Option 2?'"/>
		<xsl:with-param name="child">
			<xsl:call-template name="form:radio">
				<xsl:with-param name="handle" select="'option'"/>
				<xsl:with-param name="value" select="'Option 2'"/>
			</xsl:call-template>
		</xsl:with-param>
		<xsl:with-param name="child-position" select="'before'"/>
	</xsl:call-template>

### form:select
Renders an HTML `select` element. Has several presets to build commonly-used sets of options, and supports many formats to create additional options.

#### Parameters

* `handle` (mandatory, string): Handle of the field name
* `options` (mandatory, XPath/XML): Options to build a list of <option> elements. Has presets! See examples.
* `value` (optional, string/XML): Initial selected value
* `class` (optional, string): Class attribute value
* `title` (optional, string): Title attribute value
* `allow-multiple` (optional, string): Allow selection of multiple options ("yes", "no"). Defaults to "no"
* `section` (optional, string): Use with EventEx to change "fields[...]" to a section handle
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node

#### Examples

	<xsl:call-template name="form:select">
		<xsl:with-param name="handle" select="'categories'"/>
		<xsl:with-param name="options" select="/data/countries/country"/>
		<xsl:with-param name="value" select="'United Kingdom'"/>
	</xsl:call-template>
	
	<xsl:call-template name="form:select">
		<xsl:with-param name="handle" select="'categories'"/>
		<xsl:with-param name="options" select="/data/countries/country"/>
		<xsl:with-param name="allow-multiple" select="'yes'"/>
		<xsl:with-param name="value">
			<value>United Kingdom</value>
			<country>Australia</country>
		</xsl:with-param>
	</xsl:call-template>

In the above example, multiple selected options can be achieved by passing XML to the `value` parameter. Only the node text value is used (no attributes).

The `options` parameter accepts only XML: an XPath expression as the `select` attribute, a child `xsl:copy-of`, hard-coded, or a combination of the latter:

	<xsl:with-param name="options">
		<option value="">Select a country:</option>
		<country>Australia</country>
		<xsl:copy-of select="/data/countries/country"/>
		<country>Zimbabwe</country>
	</xsl:with-param>

Each node passed to this parameter will be converted to an HTML `option` element. If the node has an attribute of the following names: `handle`, `id`, `link-id`, `link-handle` or `value`; these will be used as the `value` attribute of the `option` element. If none of these attributes is found, no value attribute will be appended. As in the above example, to achieve an empty value for the field (for validation, for example) add an empty value attribute to an option.

The `options` parameter also accepts pre-defined string values as aliases for commonly-used sets of options.

##### Days

	<xsl:with-param name="options" select="'days'"/>
	<option>1</option>...<option>31</option>

##### Months

	<xsl:with-param name="options" select="'months'"/>
	<option value="01">January</option>...<option value="12">December</option>

##### Years (+/-)

	<xsl:with-param name="options" select="'years+20'"/>
	<option>2009</option>...<option>2029</option>
	
	<xsl:with-param name="options" select="'years-5'"/>
	<option>2009</option>...<option>2004</option>

### form:radiobutton-list
Renders a collection of HTML radio `input` elements wrapped with `label` elements. Used as a replacement for single-selection `select` elements.

#### Parameters
* `handle` (mandatory, string): Handle of the field name
* `options` (mandatory, XPath/XML): Options to build a list of <option> elements. Has presets! See examples.
* `value` (optional, string): Initial selected value
* `class` (optional, string): Class attribute value
* `title` (optional, string): Title attribute value
* `section` (optional, string): Use with EventEx to change "fields[...]" to a section handle
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node

#### Example

	<xsl:call-template name="form:radiobutton-list">
		<xsl:with-param name="handle" select="'country'"/>
		<xsl:with-param name="options" select="/data/countries/country"/>
		<xsl:with-param name="value" select="'United Kingdom'"/>
	</xsl:call-template>

### form:checkbox-list
Renders a collection of HTML checkbox `input` elements wrapped with `label` elements. Used as a replacement for allow-multiple `select` elements.

#### Parameters
* `handle` (mandatory, string): Handle of the field name
* `options` (mandatory, XPath/XML): Options to build a list of <option> elements. Has presets! See examples.
* `value` (optional, string/XML): Initial selected value
* `class` (optional, string): Class attribute value
* `title` (optional, string): Title attribute value
* `section` (optional, string): Use with EventEx to change "fields[...]" to a section handle
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node

#### Example

	<xsl:call-template name="form:radiobutton-list">
		<xsl:with-param name="handle" select="'country'"/>
		<xsl:with-param name="options" select="/data/countries/country"/>
		<xsl:with-param name="value" select="'United Kingdom'"/>
	</xsl:call-template>

Multiple selected options can be achieved by passing XML to the `value` parameter (see `form:select` example).

### form:validation-summary
Renders a success/error message and list of invalid fields.

#### Parameters
* `error-message` (optional, string/XPath): Error notification message. Defaults to Symphony Event message
* `success-message` (optional, string/XPath): Success notification message. Defaults to Symphony Event message
* `erorrs` (optional, XML): Custom error messages for individual fields as <error> nodes. Defaults to Symphony Event defaults
* `section` (optional, string): Use with EventEx to show errors for a specific section handle only
* `event` (optional, XPath): XPath expression to the specific event within the page <events> node

#### Example

	<xsl:call-template name="form:validation-summary"/>
	
	<xsl:call-template name="form:validation-summary">
		<xsl:with-param name="success-message" select="'The entry was saved.'"/>
		<xsl:with-param name="error-message" select="'The entry was not saved because of the following errors:'"/>
		<xsl:with-param name="errors">
			<error handle="title">Post Title contained an error</error>
			<error handle="email-address" type="missing">Please enter your e-mail address</error>
			<error handle="email-address" type="invalid">Please enter a valid e-mail address</error>
			<error handle="content" type="missing,invalid">Post Content is missing or invalid</error>
		</xsl:with-param>
	</xsl:call-template>

## Multiple forms per page
* Passing $event as a parameter rather than a global variable

## Submitting to multiple sections (EventEx)
* Using EventEx, section handles
* Validation summary for a specific section

## Building a form automatically (Section Schemas)
* Using Section Schemas
* Forthcoming `form:build-control` template