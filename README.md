tphases
=======

TPhases (Transactional Phases) is a support framework that helps you build your Rails request life cycles into read-only and write-only phases.

The way it accomplishes this is with the methods `TPhases.read_phase`, `TPhases.write_phase` and `TPhases.no_transactions_phase` which take blocks.  Here is a simple example inside of a controller action:

```ruby
class BarsController < ApplicationController

  def update
    TPhases.read_phase do
      @bar = Bar.find(params[:id])
    end
    
    TPhases.write_phase do
      @bar.update_attributes(params[:bar])
    end

    TPhases.no_transactions_phase do
      process(@bar)
      redirect_to @bar
    end
  end
end
```

## What it does:

### In production:
The `read_phase`, `write_phase`, and `no_transaction_phase` methods simply yield to the block given.  

### In development:

#### `read_phase`
throws an exception if a database transaction is attempted within its block which is a write.  This is known as a "read transactional violation".

#### `write_phase`
throws an exception if a database transaction is attempted within its block which is a read.  This is a write transactional violation.

#### `no_transactions_phase`
throws an exception if any database transaction is attempted within its block.

### In test:
If a transactional violation occurs in a TPhase, the code will continue to run, but the test will fail at the end citing the list of transactional violations that occurred.
