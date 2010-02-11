!SLIDE bullets incremental

# ¿Por qué no utilizo fixtures? #

* Elementos globales
* Las fixtures están dispersas
* Están alejadas de los test

!SLIDE subsection incremental

# factory_girl #

### gem install factory_girl --source http://gemcutter.org ###


!SLIDE ruby small

# Creando datos de test #

    @@@ ruby
    Factory.sequence(:login) { |n| "login#{n}" } # Sequence

    Factory.define :user do |u|
      u.login { Factory.next(:login) } # Using sequence
      u.password {|a| a.login + "pass"} # Dependent attributes
      u.password_confirmation {|a| a.login + "pass"}
      u.email {|a| "#{a.login}@example.com".downcase }
    end

    >>Factory(:user)
    +----+--------+--------------------+---------+-------+
    | id | login  | email              | state   | admin |
    +----+--------+--------------------+---------+-------+
    | 1  | login1 | login1@example.com | passive | false |
    +----+--------+--------------------+---------+-------+


!SLIDE ruby small

# Herencia #

    @@@ ruby
    Factory.define :admin, :parent => :user do |a|
      a.admin true
    end

    >>Factory(:admin)
    +----+--------+--------------------+---------+-------+
    | id | login  | email              | state   | admin |
    +----+--------+--------------------+---------+-------+
    | 3  | login3 | login3@example.com | passive | true  |
    +----+--------+--------------------+---------+-------+


!SLIDE ruby tiny

# Callbacks #

    @@@ ruby
    Factory.define :registered, :parent => :user do |m|
      m.after_create { |m| m.register! }
    end

    >>Factory(:registered)
    +----+--------+--------------------+------------------------------------------+---------+-------+
    | id | login  | email              | activation_code                          | state   | admin |
    +----+--------+--------------------+------------------------------------------+---------+-------+
    | 7  | login5 | login5@example.com | a6ea204bac5a3122cfa496656196b22e4c24b9b8 | pending | false |
    +----+--------+--------------------+------------------------------------------+---------+-------+

    Factory.define :member, :parent => :registered do |m|
      m.after_create { |m| m.activate! }
    end

    >>Factory(:member)
    +----+--------+-------------------------+--------+-------+
    | id | login  | activated_at            | state  | admin |
    +----+--------+-------------------------+--------+-------+
    | 9  | login7 | 2010-01-31 17:00:49 UTC | active | false |
    +----+--------+-------------------------+--------+-------+


!SLIDE ruby smaller

# Miss Machinist or ObjectDaddy? #

    @@@ ruby
    require 'factory_girl/syntax/generate'

    User.generate(:name => 'Johnny') # Factory(:user, :name => 'Johnny')

    require 'factory_girl/syntax/make'

    User.make(:name => 'Johnny') # Factory(:user, :name => 'Johnny')


!SLIDE ruby tiny

# Varios #

    @@@ ruby
    Factory.define :user do |u|
      u.activation_code { User.make_activation_code } # Lazy attributes
      u.email {|a| "#{a.login}@example.com".downcase } # Dependant attributes 
    end

    Factory.define :group do |p|
      # ...
      p.author {|a| a.association(:user, :login => 'Writely') } # Associations
      p.association :author, :factory => :member # o así, si necesitamos un member
    end

    >>Factory(:group)
    +----+-------+---------+---------+
    | id | name  | state   | user_id |
    +----+-------+---------+---------+
    | 2  | name2 | pending | 11      |
    +----+-------+---------+---------+

    >>User.find(11)
    +----+---------+---------------------+-------------------------+--------+
    | id | login   | email               | activated_at            | state  |
    +----+---------+---------------------+-------------------------+--------+
    | 11 | login11 | login11@example.com | 2010-01-31 17:15:37 UTC | active |
    +----+---------+---------------------+-------------------------+--------+


