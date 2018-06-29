# Postgres-Backed Enums

Rails supports soft [Enums](http://guides.rubyonrails.org/v5.2.0/active_record_querying.html#enums) but doesn't enforce data integrity on the database level. Luckily, we can make use of Postgres' Data Types for that:

```ruby
class AddFavoriteSeasonToPeople < ActiveRecord::Migration[5.2]
  def up
    execute "CREATE TYPE season AS ENUM ('spring', 'summer', 'fall', 'winter');"
    add_column :people, :favorite_season, :season
  end

  def down
    remove_column :people, :favorite_season
    execute 'DROP TYPE season'
  end
end

class Person < ApplicationRecord
  enum favorite_season: {
    spring: 'spring',
    summer: 'summer',
    fall: 'fall',
    winter: 'winter'
  }
end

person = Person.first

# happy path
person.update!(favorite_season: 'summer')
person.favorite_season # => 'summer'

# soft validation
person.update!(favorite_season: 'not a season') # => ArgumentError: 'not a season' is not a valid favorite_season

# hard constraint
person.update_column(:favorite_season, 'not a season') # => ActiveRecord::StatementInvalid: PG::InvalidTextRepresentation: ERROR:  invalid input value for enum favorite_season: "not a season"
```
