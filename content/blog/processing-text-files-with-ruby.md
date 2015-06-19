---
layout: post
title: "Processing Text Files with Ruby"
date: 2012-04-14T16:30:00
comments: true
categories: ruby
---

I occasionally find myself needing to modify a text file line-by-line. I've used sed and awk many times in the past, but not often enough to avoid having to read documentation every time I use them. Even if I did use them more regularly, performing advanced operations using either of these tools would still prove challenging. On the other hand, Ruby is something I use almost daily and it's significantly more expressive and powerful than either of these tools. As a result, using Ruby for text processing tasks is a natural fit.

Here's the boilerplate I'm currently using to do just that:

``` ruby
#!/usr/bin/env ruby
require 'fileutils'
require 'tempfile'

# Create a temporary file for the modifications.
temp_file = Tempfile.new('appended')

# Step through the source file, line by line.
File.open(ARGV[0], 'r').each_line do |line|
  # Do something awesome.
end

# Close the temporary file.
temp_file.close

# Replace the source file with the temporary file.
FileUtils.mv temp_file.path, ARGV[0]
```

The boilerplate script will operate on whatever file you've specified as an argument. If you're curious about the first/shebang line of this script, read my post about [writing executable ruby scripts](/blog/writing-executable-ruby-scripts/ "Writing Executable Ruby Scripts"). You'll also notice that it writes modifications to a temporary file and then replaces the original file with it once everything's kosher. This will cover us in case our script fails in the middle of processing.

Finally, here's a finished script using the boilerplate that I've written to do something rather simple: append a specified character to every line in a file.

``` ruby
#!/usr/bin/env ruby
require 'fileutils'
require 'tempfile'

# Create a temporary file for the modifications.
temp_file = Tempfile.new('appended')

# Step through the source file, line by line.
File.open(ARGV[0], 'r').each_line do |line|
  # Append the specified character to the line and write it to the temporary file.
  temp_file.puts line.sub "\n", ARGV[1]
end

# Close the temporary file.
temp_file.close

# Replace the source file with the temporary file.
FileUtils.mv temp_file.path, ARGV[0]
```

Move this script into a directory in your path and you can call ``append filename.txt ";"`` to insert ugly semi-colons after every line in your file. Isn't that just wonderful?
