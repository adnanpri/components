## A Form Base-Model for Laravel

Forms are often used to interact with a specific model such as a user or a blog post. However, in many circumstances a form may collect data that is related to multiple data models. Consequently, may also have special validation requirements that have little to do with the underlying data, such as captcha and password confirmation. Consequently, it often makes sense to create a form model. A form model represents the data needs of a form. This may be validation alone, storing values for form select drop-downs, having custom methods to generate data for the form, or managing persistent data in a session to make multi-page forms simple.

This form base-model is currently in development. It is very likely that this class will be continuously refactored over time to support more use-cases. This project is open for community participation so please feel free to submit issues and/or pull-requests.

### Feature Overview

- Form validation mechanism.
- Persistent form-data management for multi-page forms.
- Simple form field re-population.
- Interface for accessing persistent form-data.

### Installation

Install with artisan

	php artisan bundle:install form-base-model

or, clone the project into **bundles/form-base-model**.

Then, update your bundles.php to auto-start the bundle.

	return array(
		'form-base-model' => array( 'auto' => true ),
	);

**Notice:** The form-base-model bundle comes with examples that will auto-route to http://yoursite/form_examples. Remember to remove this from bundles/form-base-model/start.php when you're finished with the examples.

### Examples

**Example form model:**

	class ExampleForm extends FormBase_Model
	{

		public static $rules = array(
			'first_name' => 'required',
			'status'     => 'required',
		);

		public static $status = array(
			'' => 'Choose One',
			1  => 'Active',
			2  => 'inactive',
			3  => 'Deleted',
			4  => 'Boring',
			5  => 'Adventurer',
		);

		public static $foods = array(
			1 => 'Tacos',
			2 => 'Chicken Tacos',
			3 => 'Beef Tacos',
			4 => 'Vegetarian Tacos',
			5 => 'Black Bean Nachos',
		);

	}

**Simple example controller usage:**

	// simple form example

	public function get_simple_example()
	{

		return View::make( 'form-base-model::simple_example' );

	}

	public function post_simple_example()
	{

		if( ExampleForm::is_valid() )
		{
			
			return Redirect::to_route( 'form_examples', array( 'simple_example_review' ) );

		}
		else
			return Redirect::back()->with_input()->with_errors( ExampleForm::$validation );

	}

**Multi-page form example controller usage:**

	public function get_multi_page_example()
	{

		// clear any lingering persistent data as we generate a new form instance

		ExampleForm::forget_input();

		return Redirect::to_route( 'form_examples', array( 'multi_page_example_one' ) );

	}

	public function get_multi_page_example_one()
	{

		return View::make( 'form-base-model::multi_page_example_one' );

	}

	public function post_multi_page_example_one()
	{

		// define which fields this request will validate and save

		$fields = array( 'first_name', 'last_name', 'status' );

		// validate form and redirect with errors on failure

		if( ExampleForm::is_valid( $fields ) )
		{
		
			// save input to session

			ExampleForm::save_input( $fields );
			
			// redirect to the next page

			return Redirect::to_route( 'form_examples', array( 'multi_page_example_two' ) );

		}
		else
			return Redirect::back()->with_input()->with_errors( ExampleForm::$validation );

	}
	
	public function get_multi_page_example_two()
	{

		return View::make( 'form-base-model::multi_page_example_two' );

	}

	public function post_multi_page_example_two()
	{
		
		$fields = array( 'street_address', 'suite_number', 'favorite_foods' );

		if( ExampleForm::is_valid( $fields ) )
		{
			
			ExampleForm::save_input( $fields );
			
			return Redirect::to_route( 'form_examples', array( 'multi_page_example_review' ) );

		}
		else
			return Redirect::back()->with_input()->with_errors( ExampleForm::$validation );

	}


**Populating form views:**

The populate() method will fill a form field with the value stored in the persistent form data. It supports an optional second parameter that represents the default value which should be returned when the requested field data is empty. If input flash data exists (should there be validation errors and the user is redirected to the form to correct them) then the flash data will be used instead.

	First Name: {{ Form::text( 'first_name', ExampleForm::populate( 'first_name' ) ) }}

**Retrieve form data for processing**

The get() method returns the data for the requested form field and supports and optional second parameter that represents the default value which should be returned when the requested field data is empty.

	ExampleForm::get( 'status', 'none entered' );

**Note:** It's important to point out that get() relies on data saved with save_input(). If you aren't using the persistent data just use Input::get();

The all() method returns an array of all field data.

	ExampleForm::all();

To clear persistent data from a form model just tell it to forget the input.

	ExampleForm::forget_input();
	
**Saving form data to Eloquent model**

When it's time to save the data from your form to your database just access the form the same way you'd access data using Laravel's Input class. 

	$user = new User;
	
	$user->first_name = ExampleForm::get( 'first_name' );
	$user->last_name  = ExampleForm::get( 'last_name' );
	
	$user->save();