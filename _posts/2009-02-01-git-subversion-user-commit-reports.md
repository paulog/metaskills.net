--- 
layout: post
title: Git & Subversion User Commit Reports
disqus_id: /2009/02/01/git-subversion-user-commit-reports/
categories: 
  - heuristics
  - miscellaneous
  - ruby-rails
  - workflow
---


<p>
  Want a list of the users and the number of commits they made? Git makes it really really easy, while I could not find such an easy method on Subversion. Here they are.
</p>


<h2>Git</h2>

<pre class="command">
$ git log | git shortlog -n -s
</pre>


<h2>Subversion</h2>

~~~ruby
#!/usr/bin/env ruby

require 'rubygems'
require 'activesupport'

log_xml = `svn log -q --xml`
svn_logs = XmlSimple.xml_in(log_xml)['logentry']

report_hash = svn_logs.inject({}) do |report,log|
  author = log['author'][0]
  report[author] ||= {:commit_count => 0}
  report[author][:commit_count] += 1
  report
end

commits_authors = report_hash.keys.map { |a| [report_hash[a][:commit_count], a] }
commits_authors = commits_authors.sort_by(&:first).reverse

cc_colsize = commits_authors.map(&:first).max.to_s.size
a_colsize = commits_authors.map(&:last).inject{|m,w| m.length > w.length ? m : w }.size

final_report = commits_authors.map do |ca|
  c,a = ca
  " #{c.to_s.rjust(cc_colsize)}  #{a.ljust(a_colsize)}"
end.join("\n")

puts final_report
~~~



