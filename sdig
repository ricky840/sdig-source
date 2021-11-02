#!/usr/bin/env ruby

require 'optparse'
require 'ipaddr'

def ipaddr?(ipaddr)
  begin
    IPAddr.new ipaddr
    return true
  rescue IPAddr::Error, IPAddr::InvalidAddressError
    return false
  end
end

def digCmd(domain, nameserver = nil)
	output = ""

  if not nameserver == nil
    output = %x[dig @#{nameserver} #{domain} +short]
  else
    output = %x[dig #{domain} +short]
  end

  return output
end

def runDig(domain, nameserver)
  result = { :output => "", :chain => [], :ip => "" }

	output = digCmd(domain, nameserver)
	result[:output] = output

  if not output.empty?
    chain = output.split("\n")
		result[:chain] = chain

    chain.each do |each_line|
      if ipaddr? each_line
				result[:ip] = each_line.strip.chomp
      end
    end
  end

	return result 
end

def getStagingDomain(store)
	output = store[:lookup_chain]
	staging_domain = ""

 	output.each do |each|
    if each =~ /.*akamaiedge\.net\./ or each =~ /.*akamai\.net\./
      staging_domain = each.to_s.gsub(".net.", "-staging.net.")
    end
  end

	return staging_domain
end

def lookUp(domain, options)
	store = {
		:dig_output => "",
		:lookup_chain => [],
		:ip => "",
		:staging_domain => "",
		:staging_ip => ""
	}	

	production_result = runDig(domain, options[:nameserver])

	store[:dig_output] = production_result[:output]
	store[:lookup_chain] = production_result[:chain]
	store[:ip] = production_result[:ip]

	if not store[:ip].empty? and ipaddr? store[:ip]
		store[:staging_domain] = getStagingDomain(store)
		staging_result = runDig(store[:staging_domain], options[:nameserver])
		store[:staging_ip] = staging_result[:ip]
	end

	return store
end

options = Hash.new

optparse = OptionParser.new do |opts|
  opts.banner = "Usage: sdig www.foo.com [options]"

  opts.on('-v', '--verbose', 'Verbose output. Show whole resolution chain.') do
    options[:verbose] = true
  end

	opts.on('-n', '--nameserver NAMESERVER', 'Use this nameserver to resolve.') do |dns|
		if !ipaddr?(dns)
			puts "sdig: nameserver address is in a wrong format."
			exit
		end
    options[:nameserver] = dns
  end

  opts.on('-a', '--add', 'Add staging IP spoofing to the hosts file.') do
    options[:add] = true
  end

  opts.on('-r', '--remove', 'Delete all spoofing entries for the domain from the hosts file.') do
    options[:remove] = true
  end

  opts.on('-e', '--etn NUMBER(1~11)', 'Add ETN server spoofing to hosts file.') do |number|
    if not number.to_i.between?(1,11)
      puts "Number should be between 1 ~ 11"
      exit
    end
    options[:etn] = number
  end

  opts.on('-h', '--help', 'Display help message.') do
    puts opts
    exit
  end
end

begin
  optparse.parse!
  if ARGV.length.eql? 0
    raise OptionParser::MissingArgument
  end
rescue OptionParser::ParseError => e
  # puts e.message
  puts optparse
  exit
end

target_domain = ARGV[0].strip

# Remove entry from /etc/hosts and exit
if options[:remove]
  cmd = %Q[sudo sh -c 'sed -i "" "/.*#{target_domain}/d" /etc/hosts']
  if system(cmd)
    puts "sdig: removed #{target_domain} from /etc/hosts."
  else
    puts "sdig: oops something went wrong."
  end
	exit
end

# ETN option
if options[:etn]
  etn = options[:etn]
  etn_ip = %x[dig etn#{etn}.akamai.com +short].split("\n").first
  cmd = "sudo sh -c 'echo #{etn_ip.strip} #{target_domain} >> /etc/hosts'"
  if system(cmd)
    puts "sdig: etn#{etn}.akamai.com(#{etn_ip}) was added to /etc/hosts."
  else
    puts "sdig: oops something went wrong."
  end
	exit
end

# Resolve
result = lookUp(target_domain, options)

# Domain is not resolvable
if result[:ip].empty? && !ipaddr?(result[:ip])
	puts "sdig: domain is not resolvable. '#{target_domain}'"
	exit
end

# Domain is akamaized and has staging ip
if !result[:staging_ip].empty? && ipaddr?(result[:staging_ip])
	if options[:verbose]
		puts result[:lookup_chain]
	end

	puts result[:staging_domain]
	puts result[:staging_ip]
end

# Domain is not akamaized
if !result[:ip].empty? && result[:staging_ip].empty?
	puts "sdig: domain is not Akamaized, showing dig output."
	puts result[:lookup_chain]
	exit
end

# Add option
if options[:add] && ipaddr?(result[:staging_ip])
	cmd = "sudo sh -c 'echo #{result[:staging_ip]} #{target_domain} >> /etc/hosts'"
	if system(cmd)
		puts "sdig: #{result[:staging_ip]} was added to /etc/hosts."
	else
		puts "sdig: oops something went wrong."
	end
end
