# motion-turbolinks-demo
A fully featured example of using Turbolinks in a RubyMotion iOS app.



Be aware to:

 1) Include Turbolinks as a Pod in your Rakefile
 pod 'Turbolinks', :git => 'https://github.com/turbolinks/turbolinks-ios.git'
 
 2) Include the class in AppDelegate.
 include Turbolinks
 
 3) use this for your first "Hello App.." xD:
 
 
 class AppDelegate < PM::Delegate
  include CDQ # Remove this if you aren't using CDQ
  include Turbolinks
  status_bar true, animation: :fade
  
  # Without this, settings in StandardAppearance will not be correctly applied
  # Remove this if you aren't using StandardAppearance
  ApplicationStylesheet.new(nil).application_setup

  def on_load(app, options)
    #open_tab_bar HomeScreen
    cdq.setup # Remove this if you aren't using CDQ
    #open HomeScreen.new(nav_bar: true)
    open_tab_bar HomeScreen, LocationsScreen, WigScreen
  end


  # Remove this if you are only supporting portrait
  def application(application, willChangeStatusBarOrientation: new_orientation, duration: duration)
    # Manually set RMQ's orientation before the device is actually oriented
    # So that we can do stuff like style views before the rotation begins
    device.orientation = new_orientation
  end
  
  attr_reader :session, :navigationController

    def application
      UIApplication.sharedApplication
    end

    def webViewConfiguration
      @webViewConfiguration ||= begin
        @webViewProcessPool ||= WKProcessPool.alloc.init
        configuration = WKWebViewConfiguration.alloc.init
        configuration.userContentController.addScriptMessageHandler(self, name: "turbolinksDemo")
        configuration.processPool = @webViewProcessPool
        configuration.applicationNameForUserAgent = "TurbolinksDemo"
        configuration
      end
    end

    def application(application, didFinishLaunchingWithOptions:launchOptions)
      @navigationController = UINavigationController.alloc.init

      @window = UIWindow.alloc.initWithFrame(UIScreen.mainScreen.bounds)
      @window.rootViewController = navigationController
      @window.makeKeyAndVisible

      @session = Turbolinks::Session.new(webViewConfiguration: webViewConfiguration)
      @session.delegate = self
      startApplication

      true
    end
    
    def startApplication
      initial_url = NSURL.alloc.initWithString("http://localhost:3000/mobile")
      presentVisitableForSession(session, URL: initial_url, action: nil) # TODO: is there a way to call this with the optional argument?
    end

    def presentVisitableForSession(session, URL: url, action: action)
      action ||= :advance
      puts "ApplicationDelegate#presentVisitableForSession:URL:action"
      visitable = Turbolinks::VisitableViewController.new(url: url)

      if action == :advance
        navigationController.pushViewController(visitable, animated: true)
      elsif action == :replace
        navigationController.popViewControllerAnimated(false)
        navigationController.pushViewController(visitable, animated: false)
      end

      session.visit(visitable)
    end

    def session(session, didProposeVisitToURL: url, withAction: action)
      puts "AppDelegate#session:didProposeVisitToURL:withAction url: #{url} action: #{action}"
      if url.path == "/numbers"
        presentNumbersViewController
      else
        presentVisitableForSession(session, URL: url, action: action)
      end
    end

    def presentNumbersViewController
      puts "AppDelegate#presentNumbersViewController"
      viewController = NumbersViewController.alloc.init
      navigationController.pushViewController(viewController, animated: true)
    end

    def presentAuthenticationController
      puts "AppDelegate#presentAuthenticationController"
      authenticationController = AuthenticationController.alloc.init
      authenticationController.delegate = self
      authenticationController.webViewConfiguration = webViewConfiguration
      authenticationController.URL = URL.URLByAppendingPathComponent("sign-in")
      authenticationController.title = "Sign in"

      authNavigationController = UINavigationController.alloc.initWithRootViewController(authenticationController)
      navigationController.presentViewController(authNavigationController, animated: true, completion: nil)
    end

    def session(session, didFailRequestForVisitable: visitable, withError: error)
      puts "AppDelegate#session:didFailRequestForVisitable:withError"
      alert = UIAlertController.alertControllerWithTitle("Error", message: error.localizedDescription, preferredStyle: UIAlertControllerStyleAlert)
      alert.addAction(UIAlertAction.actionWithTitle("OK", style: UIAlertActionStyleDefault, handler: nil))
      navigationController.presentViewController(alert, animated: true, completion: nil)
      # TODO
      # switch errorCode {
      #   case .HTTPFailure:
      #       let statusCode = error.userInfo["statusCode"] as! Int
      #       switch statusCode {
      #       case 401:
      #           presentAuthenticationController()
      #       case 404:
      #           demoViewController.presentError(.HTTPNotFoundError)
      #       default:
      #           demoViewController.presentError(Error(HTTPStatusCode: statusCode))
      #       }
      #   case .NetworkFailure:
      #       demoViewController.presentError(.NetworkError)
      #   }
    end

    def sessionDidStartRequest(session)
      puts "AppDelegate#sessionDidStartRequest"
      application.networkActivityIndicatorVisible = true
    end

    def sessionDidFinishRequest(session)
      puts "AppDelegate#sessionDidFinishRequest"
      application.networkActivityIndicatorVisible = false
    end

    def authenticationControllerDidAuthenticate(authenticationController)
      puts "AppDelegate#authenticationControllerDidAuthenticate"
      session.reload
      navigationController.dismissViewControllerAnimated(true, completion: nil)
    end

    def userContentController(userContentController, didReceiveScriptMessage: message)
      puts "AppDelegate#userContentController:didReceiveScriptMessage"
      if message = message.body
        alert = UIAlertController.alertControllerWithTitle("Turbolinks", message: message, preferredStyle: UIAlertControllerStyleAlert)
        alert.addAction(UIAlertAction.actionWithTitle("OK", style: UIAlertActionStyleDefault, handler: nil))
        navigationController.presentViewController(alert, animated: true, completion: nil)
      end
    end
  
  
end

 
