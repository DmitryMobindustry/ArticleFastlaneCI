default_platform(:ios)

platform :ios do

### ----- TESTS ----- ###
	lane :run_tests_project do 
		begin
	       run_tests(devices: ["iPhone 6s"], scheme: "TestAppTests")
	    rescue => exception
	       on_error(exception)
	    end
	end


### ----- BUILDS ----- ###
	desc "Builds new version with Debug configuration. Bumps build number."
	lane :build_dev do
		increment_build_number(build_number: ENV["CI_PIPELINE_ID"])
		build_configuration("Debug", "development", "TestApp-Dev.ipa")
	end

	desc "Builds new version with Stage configuration. Bumps build number."
	lane :build_stage do
		increment_build_number(build_number: ENV["CI_PIPELINE_ID"])
		build_configuration("Stage", "ad-hoc", "TestApp-Stage.ipa")
	end	

	desc "Builds new version with Release configuration. Bumps build number."
	lane :build_release do
		increment_build_number(build_number: ENV["CI_PIPELINE_ID"])
		build_configuration("Release", "app-store", "TestApp.ipa")
	end

### ----- EXPORTS ----- ###

	desc "Uploads build with Debug configuration to Crashlytic."
	lane :export_dev do
		crashlytic_upload "Artifacts/TestApp-Dev.ipa"
	end

	desc "Uploads build with Stage configuration to Crashlytic."
	lane :export_stage do
		testflight_upload "Artifacts/TestApp-Stage.ipa"
		crashlytic_upload "Artifacts/TestApp-Stage.ipa"
	end

	desc "Uploads build with Release configuration to Testflight."
	lane :export_release do
		testflight_upload "Artifacts/TestApp.ipa"
	end

### ----- METHODS ----- ###

	def crashlytic_upload(ipa_path)
		begin
            crashlytics(
			  crashlytics_path: "./Pods/Crashlytics/submit", # path to your Crashlytics submit binary.
			  api_token: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
			  build_secret: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
			  ipa_path: ipa_path,
			  emails: "TestUser1@mobindustry.net,TestUser2@mobindustry.net,TestUser3@mobindustry.net,TestUser4@mobindustry.net",
			)
       rescue => exception
           on_error(exception)
       end
	end

	def build_configuration(config, export_method, output_name)
 	 	begin
 	 		disable_automatic_code_signing(path: "TestApp.xcodeproj")
            build_ios_app(
		 		 workspace: "TestApp.xcworkspace",
			 	 configuration: config,
			 	 scheme: "TestApp",
			 	 export_method: export_method,
			 	 output_directory: "Artifacts", # Destination directory. Defaults to current directory.
			 	 output_name: output_name, # specify the name of the .ipa file to generate (including file extension)
 	 		 )
       rescue => exception
           on_error(exception)
       end
	end

	def testflight_upload(ipa_path)
		begin
			ENV["FASTLANE_DONT_STORE_PASSWORD"] = "1"
	    	ENV["FASTLANE_USER"] = ENV["APP_STORE_LOGIN"]
	    	ENV["FASTLANE_PASSWORD"] = ENV["APP_STORE_PASSWORD"]
            upload_to_testflight(
			  	username: "username@mobindustry.net",
			  	app_identifier: "net.mobindustry.testapp",
			  	ipa: ipa_path,
			  	skip_submission: true,
				skip_waiting_for_build_processing: true
			)
       rescue => exception
           on_error(exception)
       end
	end

	def on_error(exception)
       slack(
           message: "Lane failed with exception : #{exception}",
           success: false,
           slack_url: "https://hooks.slack.com/services/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
           attachment_properties: {
	           fields: [
	               {
	                   title: "Build number",
	                   value: get_build_number(xcodeproj: "TestApp.xcodeproj"),
	               },
	               {
	                   title: "Version number",
	                   value: get_version_number(xcodeproj: "TestApp.xcodeproj", target: "TestApp"),
	               }
	           ]
	       }
       )
       raise exception
	end
end