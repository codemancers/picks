## Harshwardhan

### Form Objects
Decoupling the methods that are there only for handling forms from your model to another class is a popular practice while refactoring the rails
application. It moves the logic into form object class and makes your model skinny.

Form objects are the pure ruby classes which are created while refactoring the rails application. Form object are important because they are used to back and process the form data submitted by the user. Form objects are responsible to handling the data of a specific feature. A Form object encapsulates the logic for validating incoming data and taking appropriate actions.

```
class LeaveForm
  include ActiveModel::Model

  attr_accessor(:summary, :description, :start, :end)

  validates :summary, presence: true
  validates :description, presence: true
  validates :start, presence: true
  validates :end, presence: true

  def add_leave
    if valid?
      *code_for_adding_leave*
    end
end
```

### Service objects:

Service objects are the classes each of which have a single unique purpose. Unlike putting everything in the model, service objects put them in separate classes which are assigned a specific responsibility.
This approach is used to keep the all of our code separate and organized.

```
class GoogleCalendarService
  require 'base64'

  def initialize(oauth_token, refresh_token)
    @client = Google::APIClient.new(:auto_refresh_token => true)
    @client.authorization.access_token = oauth_token
    @client.authorization.refresh_token = refresh_token
    @client.authorization.client_id = ENV["GOOGLE_CLIENT_ID"]
    @client.authorization.client_secret = ENV["GOOGLE_SECRET"]
    if @client.authorization.refresh_token && @client.authorization.expired?
      @client.authorization.fetch_access_token!
    end
    return @client
  end

  def list_calendars
    @client.execute(api_method: service.calendar_list.list,
      parameters: {'calendarId': 'primary'},
      'showDeleted': false,
      'maxResults': 10,
      headers: { 'Content-Type': 'application/json' })
  end

  def create_calendar(calendar)
    @client.execute(api_method: service.calendars.insert,
      body: JSON.dump(calendar),
      headers: {'Content-Type': 'application/json'})
  end

  def show_calendar(calendar_id)
    calendarId = Base64.urlsafe_decode64(calendar_id)
    @client.execute(api_method: service.events.list,
      parameters: { 'calendarId': calendarId,
        'timeMin': Time.now.beginning_of_month.iso8601,
        'showDeleted': false,
        'singleEvents': true,
        'maxResults': 10,
        'orderBy': 'startTime'},
      headers: { 'Content-Type': 'application/json' })
  end

  def edit_calendar(calendar_id)
    calendarId = Base64.urlsafe_decode64(calendar_id)
    @client.execute(api_method: service.calendars.patch,
      parameters: {'calendarId': calendarId})
  end

  def calendar_permissions(calendar_id)
    calendarId = Base64.urlsafe_decode64(calendar_id)
    @client.execute(api_method: service.acl.list,
      parameters: {'calendarId': calendarId})
    end

  def invite_user(calendar_id, email_id)
    calendarId = Base64.urlsafe_decode64(calendar_id)
    invite = service.acl.insert.request_schema.new(
              scope: {
                type: 'user',
                value: email_id,
              },
              role: 'writer'
              )
    @client.execute(api_method: service.acl.insert,
      parameters: {'calendarId': calendarId},
      body_object: invite)
  end

  def remove_user(calendar_id, acl_id)
    calendarId = Base64.urlsafe_decode64(calendar_id)
    aclId = Base64.urlsafe_decode64(acl_id)
    @client.execute(api_method: service.acl.delete,
      parameters: {'calendarId': calendarId,
        'ruleId': aclId})
  end
end
```
