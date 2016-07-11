## Harshwardhan

### Form Objects
Decoupling the methods that are there only for handling forms from your model to another class is a popular practice while refactoring the rails
application. It moves the logic into form object class and makes your model skinny.

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

Detailed post here https://www.reinteractive.net/posts/158-form-objects-in-rails
