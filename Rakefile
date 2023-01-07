#!/usr/bin/env ruby
require 'rake'

require 'csv' # https://rubygems.org/gems/csv
require 'yaml'

task default: %w(import)

task :import => %w(goodreads.yaml)

file 'goodreads.yaml' => %w(goodreads_library_export.csv) do |t|
  File.open(t.name, 'w') do |file|
    line_no = 0

    CSV.foreach(t.prerequisites.first, headers: true) do |row|
      line_no += 1

      # Clean up the data from Goodreads:
      title     = row['Title'].strip
      author    = row['Author'].strip
      authors   = row['Additional Authors'].strip.split(',') rescue nil
      authors   = authors.map{ |s| s.strip.gsub(/\s+/, ' ') } if authors
      isbn      = row['ISBN'].gsub(/="([^"]*)"/, '\1').strip
      isbn      = nil if isbn.empty?
      isbn      = "978#{isbn}" if isbn
      isbn13    = row['ISBN13'].gsub(/="([^"]*)"/, '\1').strip
      isbn13    = nil if isbn13.empty?
      published = row['Original Publication Year'].strip rescue nil
      published = published.to_i if published
      publisher = row['Publisher'].strip rescue nil
      rating    = row['My Rating'].strip.to_i
      review    = row['My Review'].strip rescue nil
      tags      = row['Bookshelves'].strip.split(',').map(&:strip).sort rescue nil

      # Remove any personal tags:
      %w(currently-reading gave-up-on priority read shortlist to-read to-reread)
        .each { |t| tags.delete(t) } if tags

      # Publish the book record:
      file.puts if line_no > 1
      file.puts "--- !book"
      file.puts "title: #{title.inspect}"
      file.puts "author: #{author.inspect}"
      file.puts "authors: #{authors.inspect}" if authors
      file.puts "isbn: #{(isbn13 || isbn).inspect}" if isbn13 || isbn
      file.puts "published: #{published.inspect}" if published
      file.puts "publisher: #{publisher.inspect}" if publisher
      file.puts "tags: #{tags.inspect}" if tags && !tags.empty?

      # Publish the review record, if any:
      next unless rating > 0
      # TODO: date, rating, review, my_tags
    end
  end
end
