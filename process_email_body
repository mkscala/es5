import json,codecs,base64
from bs4 import BeautifulSoup
import re,time
from datetime import date

today = date.today()
 
val  = []
whitelist = []
data = {}
 
with open('csdw_email_transcript.20170112.dat') as f:
    for line in f:
        val=line.split("\001")
        sr=val[0]
	channel_name = val[2]
        opendt=val[8]
	close_dt = val[9]
	employee_staffgroup = val[18]
	intended_staffgroup = val[19]
        eml=base64.b64decode(val[43])
        soup = BeautifulSoup(eml,"html5lib")
        for tag in soup.findAll(True):
            if tag.name not in whitelist:
                tag.append(' ')
                tag.replaceWithChildren()
        result= unicode(soup)
    # Clean up any repeated spaces and spaces like this: '<a>test </a> '
        result = re.sub(r'\'','', result)
        result = re.sub(r'\,','', result)
        result = re.sub(r'\"','', result)
        data['sr']=sr
	data['channel'] = channel_name
	data['closedt'] = close_dt
        data['opendt']=opendt
	data['empstaff'] = employee_staffgroup
	data['intstaff'] = intended_staffgroup
        data['email']=result
      #  print unicode(data)
 
        with codecs.open("csdw_email_transcript.20170112.json", 'ab', 'utf8') as f:
             f.write(json.dumps(data, sort_keys = True, ensure_ascii=False))
             f.write('\n')
