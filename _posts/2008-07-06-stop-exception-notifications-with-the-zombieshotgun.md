--- 
layout: post
title: Stop Exception Notifications With The ZombieShotgun
disqus_id: /2008/07/06/stop-exception-notifications-with-the-zombieshotgun/
categories: 
  - heuristics
  - ruby-rails
---

<p>
  <span class="photofancy floatr ml20"><img src="/assets/zombie_shotgun.jpg" alt="Resident Evil Zombie Shotgun" width="320" height="213" /></span> I am all about knowing how to survive a zombie invasion â€“ as much as I am a firm believer of using the right tool for a given job. It can not be argued that killing zombies with a shotgun to the head is as natural a fit as peanut butter to chocolate. They simply just go together. 
</p>

<p>
  Now real zombies may not be a daily nuisance, but <a href="http://en.wikipedia.org/wiki/Zombie_computer">computer zombies</a> are a daily pain in the butt to network administrators as well as software engineers alike. If you have ever deployed a rails application into production that used some sort of exception notification, then you may at some time seen some zombie attacks throw a bunch of exceptions. My solution a few years back was to build my own ZombieShotgun module, see below.
</p>

<p>
  The idea is simple, include the module and add this line to the very top of your filter chain <code>before_filter :shoot_zombies</code>. Just like in real life, if rails detects a zombie attack, it will issue a 404 not found error in the beautiful rails syntax <code>head :not_found</code>. I love it when code models the real world! Please note that there are a ton of better ways to accomplish user agent filtering, most notably, in your web server config... but that does not mean this is not a fun module to use.
</p>

~~~ruby
module ZombieShotgun
  
  ZOMBIE_AGENTS       = ['Microsoft Office Protocol Discovery','Microsoft Data Access Internet Publishing Provider Protocol Discovery','FrontPage']
  ZOMBIE_ATTACK_DIRS  = ['_vti_bin','MSOffice','verify-VCNstrict','notified-VCNstrict']
  
  protected
  
  def shoot_zombies
    head :not_found if zombie_attack?
  end
  
  def zombie_attack?
    zombie_attack_on_directory? || zombie_agent_attack?
  end
  
  def zombie_attack_on_directory?
    attack = request.path.from(1)
    attack_dir = attack.index('/').nil? ? attack : attack.to(attack.index('/')-1)    
    ZOMBIE_ATTACK_DIRS.include?(attack_dir)
  end
  
  def zombie_agent_attack?
    ua = request.env['HTTP_USER_AGENT']
    !ua.blank? && ZOMBIE_AGENTS.any? { |za| za =~ /#{ua}/ }
  end
  
  
end
~~~

