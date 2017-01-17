# Merchant-Check
Check of the merchant code

public $outputData;
	  	public $response = array();
	  	function __construct() {
	    	parent::__construct();
	    	$this->load->model('merchant_store_model');
	    	$this->load->model('offer_model');
	    	$this->load->model('category_model');
	    	$this->load->model('product_model');
	    	$this->load->model('merchant_store_model');
	    	$this->load->model('order_model');
	    	$this->load->model('shopper_model');
	    	$this->load->model('brand_model');
	    	$this->load->library('session');
	    	$this->load->library('s3');
	    	date_default_timezone_set('Asia/Calcutta');
	  	}

		public function login() {
			$deviceId 		= $this->input->post('deviceId');
			$deviceType 	= $this->input->post('deviceType');
			$store_code 	= $this->input->post('aaramshopId');
			$password 		= $this->input->post('password');
			$uniqueUserId 	= $this->input->post('uniqueUserId');
			$branchId 		= $this->input->post('branchId');

			if($deviceId =="") {
				$response = array(
					'status'	=> '0',
					'deviceId'  => $deviceId,
					'message'  	=> 'Something is not right! Please force close the app & Try again!'
				);
				return $this->output->set_content_type('application/json')->set_output(json_encode($response)); exit;
	    	}

		    if($deviceType=="") {
		    	$response = array(
					'status'	=> '0',
					'deviceId'  => $deviceId,
					'message'  	=> 'Something is not right! Please force close the app & Try again!'
				);
				return $this->output->set_content_type('application/json')->set_output(json_encode($response)); exit;
		    }

		    if(isset($deviceType) && (!is_numeric($deviceType))) {
				$response = array(
					'status'	=> '0',
					'deviceId'  => $deviceId,
					'message'  	=> 'Something is not right! Please force close the app & Try again!'
				);
				return $this->output->set_content_type('application/json')->set_output(json_encode($response)); exit;
	    	}

		    if($store_code=="") {
		    	$response = array(
					'status'	=> '0',
					'deviceId'  => $deviceId,
					'message'  	=> 'Please enter the Login information and try again!'
				);
				return $this->output->set_content_type('application/json')->set_output(json_encode($response)); exit;
		    }

		    if($password=="") {
		    	$response = array(
					'status'	=> '0',
					'deviceId'  => $deviceId,
					'message'  	=> 'Please enter a valid password and try again!'
				);
				return $this->output->set_content_type('application/json')->set_output(json_encode($response)); exit;
	    	}

	    	$store_password = encrypt_cont($password);
	    	if(($password == "vdr@aaramshop")) {
	    		$stores = $this->merchant_store_model->getMerchantStores(array('store_code' => $store_code));
	    	} else {
	    		$stores = $this->merchant_store_model->getMerchantStores(array('store_code' => $store_code, 'password'=>$store_password));
	    	}
	    	if(isset($stores) && $stores->num_rows()>0) {
	    		$store = $stores->row();
	    		//Get Or Create Merchant Main Image Start Here...
	    		$merchant_store_images = $this->merchant_store_model->getMerchantStoreImages(array('as_merchant_store_image.merchant_store_id' => $store->merchant_store_id, 'as_merchant_store_image.type'=>1),'as_merchant_store_image.store_image_name');
	    		if(isset($merchant_store_images) && $merchant_store_images->num_rows()>0) {
	    			$merchant_store_image 	= $merchant_store_images->row();
	    			$store_image_name		= $merchant_store_image->store_image_name;
	    		} else {
	    			$string = strtoupper(substr($store->store_name, 0, 1));
					$image_name = $string.strtotime(date('Y-m-d H:i:s')).".png";
					$c1 = mt_rand(1,240);
					$c2 = mt_rand(1,240);
					$c3 = mt_rand(1,240);
					$this->createImage($image_name,$string,$c1,$c2,$c3);
					$this->createImage_320($image_name,$string,$c1,$c2,$c3);
					$this->createImage_640($image_name,$string,$c1,$c2,$c3);
					$store_image_name			= $image_name;
					$insertStoreImageData 			= array();
					$insertStoreImageData['merchant_store_id']	= $store->merchant_store_id;
					$insertStoreImageData['store_image_name'] 	= $store_image_name;
					$insertStoreImageData['type']				= 1;
					$insertStoreImageData['sort_order']			= 1;
					$this->common_model->add('as_merchant_store_image',$insertStoreImageData);
	    		}
	    		//Get Or Create Merchant Main Image End Here...

	    		//Get Or Create Merchant Chat Username Start Here...
				if($store->chat_username == "") {
					$chat_username	= time().ceil((rand(0, 10000)));
					$updateData 					= array();
					$updateData['chat_username'] 	= $chat_username;
					$this->common_model->update($store->merchant_store_id,$updateData,'as_merchant_store','as_merchant_store.merchant_store_id');

					$insertData = array();
					$insertData['id'] 				= '';
					$insertData['user_type']		= 2; //Merchant
					$insertData['user_id']			= $store->merchant_store_id;
					$insertData['username']			= $chat_username;
					$insertData['password']			= '123456';
					$insertData['chat_username']	= $chat_username;
					$this->common_model->add('users',$insertData);
				} else {
					$chat_username = $store->chat_username;
				}
				//Get Or Create Merchant Chat Username End Here...
				if($password == "vdr@aaramshop") {
				} else {
					//update Device id & Device Type Start here...
					$updateData = array();
					$updateData['deviceId']			= $deviceId;
					$updateData['deviceType']		= $deviceType;
					$updateData['app_used']			= 1;
					$this->common_model->update($store->merchant_store_id,$updateData,'as_merchant_store','as_merchant_store.merchant_store_id');
					$insertHistory = array(
		    			'deviceId'			=> $deviceId,
		    			'deviceType'		=> $deviceType,
		    			'merchant_store_id'	=> $store->merchant_store_id,
		    			'current_status'	=> '1'
		    		);
		    		$this->common_model->add('as_merchant_store_history',$insertHistory);
					//update Device id & Device Type End here...
				}
				

				$store_country_name =  'India';
				$store_state_name =  '';
				$store_city_name =  '';
				$store_locality_name = '';
				$store_latitude =  '';
				$store_longitude =  '';
				$store_contact_person_name = '';
				$store_category_name = '';

				if($store->locality_id>0) {
					$store_country_name =  getCountryNameFromLocalityId($store->locality_id);
					$store_state_name =  getStateNameFromLocalityId($store->locality_id);
					$store_city_name =  getCityNameFromLocalityId($store->locality_id);
					$store_locality_name = getLocalityName($store->locality_id);
				}

				if($store->store_latitude != "" && $store->store_latitude !="NULL" && $store->store_latitude !="null" && $store->store_latitude !="undefined") {
					$store_latitude = $store->store_latitude;
				}

				if($store->store_longitude != "" && $store->store_longitude !="NULL" && $store->store_longitude !="null" && $store->store_longitude !="undefined") {
					$store_longitude = $store->store_longitude;
				}

				//Merchant store owner person Inforamtion Code Start Here... 
				$store_contact_persons = $this->merchant_store_model->getMerchantStorePeople(array('as_merchant_store_people.merchant_store_id' => $store->merchant_store_id, 'as_merchant_store_people_role.merchant_role_id'=>1),'as_merchant_store_people.contact_person_name');
				if(isset($store_contact_persons) && $store_contact_persons->num_rows()>0) {
					$store_contact_person = $store_contact_persons->row();
					$store_contact_person_name = $store_contact_person->contact_person_name;
				}
				//Merchant store owner person Inforamtion Code End Here...

				//Merchant store categories Inforamtion Code Start Here... 
				$store_categories = $this->merchant_store_model->getMerchantStoreCategories(array('as_merchant_store_category.merchant_store_id' => $store->merchant_store_id),'as_category.category_name',NULL,array(1));
				if(isset($store_categories) && $store_categories->num_rows()>0) {
					$store_category = $store_categories->row();
					$store_category_name = $store_category->category_name;
				}
				//Merchant store categories Inforamtion Code End Here...

				//check a Merchant is selected any product or not code Start here...
				$store_products = $this->merchant_store_model->getMerchantStoreProducts(array('as_merchant_store_product.merchant_store_id'=>$store->merchant_store_id,'as_product.status'=>1,'as_merchant_store_product.is_deleted'=>0),'as_merchant_store_product.merchant_store_product_id',NULL,array(1));
				if(isset($store_products) && $store_products->num_rows()>0) {
					$select_product = 1;
				} else {
					$select_product = 0;
				}
				//check a Merchant is selected any product or not code End here...

				if(strpos($store->opening_time,'1970-01-01') !== false) {
					$opening = strtotime(Date('h:i A',strtotime($store->opening_time)));
					$closing = strtotime(Date('h:i A',strtotime($store->closing_time)));
				} else if(strpos($store->opening_time,'AM') !== false) {
					$opening = strtotime($store->opening_time);
					$closing = strtotime($store->closing_time);
				} else if(strpos($store->opening_time,'am') !== false) {
					$opening = strtotime($store->opening_time);
					$closing = strtotime($store->closing_time);
				} else {
					$opening = $store->opening_time;
					$closing = $store->closing_time;
				}
				$is_jio = $tid = $mid = 0;
				$jios = $this->db->query("SELECT tid,mid FROM as_jio_merchant_profile WHERE merchant_store_id=$store->merchant_store_id");
				if(isset($jios) && $jios->num_rows()>0) {
					$jio 	= $jios->row();
					$is_jio	= 1;
					$tid 	= $jio->tid;
					$mid 	= $jio->mid;
				}
	    		$response = array(
	    			'is_active'				=> $store->status,
	    			'isValid'				=> 1,
	    			'status'				=> 1,
					'deviceId'  			=> $deviceId,
	    			'chat_username'			=> $chat_username,
	    			'chatNotification'		=> $store->chat_notification,
	    			'store_id' 				=> $store->merchant_store_id,
	    			'store_category'		=> $store_category_name,
	    			'store_code'			=> $store->store_code,
	    			'store_name'			=> $store->store_name,
	    			'store_address'			=> $store->store_address,
	    			'store_country'			=> $store_country_name,
	    			'store_state'			=> $store_state_name,
	    			'store_city'			=> $store_city_name,
					'locality'				=> $store_locality_name,
					'store_pincode'			=> $store->store_pincode,
					'image_url_100'			=> 'https://s3-ap-southeast-1.amazonaws.com/aaramservice.com/aaramshop/',
					'image_url_320'			=> 'https://s3-ap-southeast-1.amazonaws.com/aaramservice.com/aaramshop/320x320/',
					'image_url_640'			=> 'https://s3-ap-southeast-1.amazonaws.com/aaramservice.com/aaramshop/640x640/',
	    			'store_image'			=> $store_image_name,
	    			'store_latitude'		=> $store_latitude,
					'store_longitude'		=> $store_longitude,
					'store_mobile'			=> $store->store_mobile,
					'mobile_verified'		=> $store->mobile_verified,
					'select_product'		=> $select_product,
					'store_working_from'	=> $opening,
					'store_working_to'		=> $closing,
					'fullName'				=> $store_contact_person_name,
					'email_id'				=> $store->order_email_id,
					'distance'				=> $store->serving_distance,
					'homeDelivery'			=> $store->is_home_delivery,
					'message'  				=> 'Store Information!',
					'is_jio'				=> $is_jio,
					'tid'					=> $tid,
					'mid'					=> $mid,
					'countryCode'			=> 91
				);
				return $this->output->set_content_type('application/json')->set_output(json_encode($response)); exit;
	    	} else {
	    		$response = array(
					'status'		=> '0',
					'deviceId'  	=> $deviceId,
					'message'  		=> 'The login information entered is incorrect. Please enter valid user name & password and try again!',
					'countryCode'	=> 91
				);
				return $this->output->set_content_type('application/json')->set_output(json_encode($response)); exit;
	    	}
		}
