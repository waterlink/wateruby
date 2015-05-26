# Wateruby

Interesting twist of ruby language: YAML contains fragments of ruby, that can be composed. Compiles to ruby. Art of true metaprogramming and code generation.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'wateruby'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install wateruby

## Usage

### Hello world example

Create `hello.wateruby`:

```yaml
language: ruby

body:
- puts "Hello, World"
```

Compiling to ruby:

```
$ wateruby hello.wateruby > hello.rb
```

Running ruby:

```
$ ruby hello.rb
Hello, World
```

### Greeter class example

```yaml
language: ruby

definitions:
  Greeter:
    define: class <%= name %> < Struct.new(:greeting)
    definitions:
      greet:
        define: def <%= name %>(name)
        body:
        - "#{greeting}, #{name}!"

body:
- puts Greeter.new("hello").greet("world")
```

### Inline method call

This is required when you are dealing with decorators for almost all methods in your system: they need to be ridicuously light - this makes method call cost very relevant, so you want to have as least as possible of them.

```yaml
language: ruby

definitions:
  Contract:
    define: class <%= name %>
    definitions:

      self.make_validator:
        define: def <%= name %>(contract)
        body:
        - klass = contract.class
        - |
          <%= inline("self.proc_contract", contract, klass) %> ||
          <%= inline("self.array_contract", contract, klass) %> ||
          <%= inline("self.hash_contract", contract, klass) %> ||
          <%= inline("self.args_contract", contract, klass) %> ||
          <%= inline("self.func_contract", contract, klass) %> ||
          <%= inline("self.default_contract", contract, klass) %>

      self.proc_contract:
        # e.g. lambda {true}
        inlinable: true
        define: def <%= name %>(contract, its_klass)
        pre: its_klass == Proc
        body: contract

      self.array_contract:
        # e.g. [Num, String]
        # TODO: account for these errors too
        inlinable: true
        define: def <%= name %>(contract, its_klass)
        pre: klass == Array
        body: |
          lambda do |arg|
            return false unless arg.is_a?(Array) && arg.length == contract.length
            arg.zip(contract).all? do |_arg, _contract|
              Contract.valid?(_arg, _contract)
            end
          end

      # and so on..
```

## Contributing

1. Fork it ( https://github.com/waterlink/wateruby/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
