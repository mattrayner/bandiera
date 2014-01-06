#!/usr/bin/env rake

libdir = File.expand_path(File.join(File.dirname(__FILE__), "lib"))
$LOAD_PATH.unshift(libdir) unless $LOAD_PATH.include?(libdir)

require "bandiera"
require "rspec/core/rake_task"

RSpec::Core::RakeTask.new

task :default => :spec
task :test    => :spec

namespace :bundler do
  task :setup do
    require "bundler/setup"
  end
end

task :environment, [:env] => "bundler:setup" do |cmd, args|
  ENV["RACK_ENV"] = args[:env] || "development"
end

namespace :db do
  desc "Create DB"
  task :create, :env do |cmd, args|
    env = args[:env] || "development"
    db  = Bandiera::Db.params(env)
    Rake::Task["environment"].invoke(env)
    run_mysql_command(db, "CREATE DATABASE `#{db[:database]}`")
  end

  desc "Drop database"
  task :nuke, :env do |cmd, args|
    env = args[:env] || "development"
    db  = Bandiera::Db.params(env)
    Rake::Task["environment"].invoke(env)
    run_mysql_command(db, "DROP DATABASE `#{db[:database]}`")
  end

  desc "Run database migrations"
  task :migrate, :env do |cmd, args|
    env = args[:env] || "development"
    Rake::Task["environment"].invoke(env)

    Sequel.extension :migration
    Sequel::Migrator.apply(Bandiera::Db.connection, "db/migrations")
  end

  desc "Rollback the database"
  task :rollback, :env do |cmd, args|
    env = args[:env] || "development"
    Rake::Task["environment"].invoke(env)

    Sequel.extension :migration
    version = (row = Bandiera::Db.connection[:schema_info].first) ? row[:version] : nil
    Sequel::Migrator.apply(Bandiera::Db.connection, "db/migrations", version - 1)
  end

  desc "Drop all tables in database"
  task :drop, :env do |cmd, args|
    env = args[:env] || "development"
    Rake::Task["environment"].invoke(env)
    Bandiera::Db.connection.tables.each do |table|
      Bandiera::Db.connection.run("DROP TABLE #{table}")
    end
  end

  desc "Clean Slate"
  task :reset, [:env] => [:nuke, :create, :drop, :migrate]
end

private

def run_mysql_command(db, cmd)
  command = "echo '#{cmd}' | mysql -h #{db[:host]} -P #{db[:port]} -u #{db[:user]} --password=#{db[:password]} 2>/dev/null"
  system(command)
end

