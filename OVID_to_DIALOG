

"""

Translating OVID Search Strategy to Dialog Search Strategy

"""

import re
import sys


def Convert_OVID_To_DIALOG(q_OVID): 
    MeSH_Dict = {}
    MeSH_List_Examples = [""]
    #replace OVID field codes & operators with Dialog field codes & operators
    process = []
    operations = [' and ' , ' or ', ' not ']
    
    for line in q_OVID:
        temp = line.strip()
        #convert field codes to Dialog format
        temp = temp.replace('.pt.','[RTYPE]').replace('.ab,ti.','[AB/TI]').replace('.ti,ab.','[TI/AB]').replace('.ti.','[TI]').replace('.ti,ab,kf','TI,AB,IF').replace('$','*').replace('.mp.','[all]').replace('.mp','[all]').replace('.ab.','[AB]').replace('.ti,ab,sh.','[all]').replace('[mp=title, original title, abstract, name of substance word, subject heading word]','')
        
        #remove number of hits from the query line
        check_hits = re.findall(r'\([0-9]+\)', temp)
        if check_hits :
            temp = temp.replace(check_hits[0],'')
        
        #remove line number  
        check_line = re.findall(r'^\d+\s+[a-zA-Z]+\s+\d+',temp)
        check_line_no =  re.findall(r'^[0-9]+\.|^[0-9]+\s+\D+|^[0-9]+\s+\d+\s+((and)|(or)|(not))\s+\d+|^#*[0-9]+\s+#*\d+\s+((and)|(or)|(not))\s+#*\d+|^#+\d+\s+|^\d+\s+\d+\s+', temp)  
        if check_line_no and not check_line:
            temp = temp.split()[1:]
            temp = ' '.join(temp)
    
                   
        #check if search contains multiple operations
        flag = 0
        for oper in operations:
            if oper in temp:
                flag = 1
        
        if flag == 1: # if line containes multiple operations
            terms = []
            op_order = []
            
            for term in temp.split():
                if (' ' + term + ' ') in operations:
                    op_order.append(term)
                    
            for term in re.split(r' or | not | and ', temp): 
                check = re.findall(r'\/[a-zA-Z]+', term)

              
            q = ''
            for i,term in enumerate(terms):
                if i < len(op_order):
                    q = q + ' ' + term + ' ' + op_order[i]
                else:
                    q = q + ' ' + term
            
            #replace adj with NEAR/n
            q = re.sub(r'adj\d*','NEAR/',q)                 
            process.append('(' + q.strip() + ' )')  
             
            
        #******************************************************************* 

def convert_to_dialog(query):
    
    q = {}
    
    for i,line in enumerate(query.split('\n')):
        q[i+1] = line
    
    #find combined operations or/1-3 convert to S1 OR S2 OR S3
    for key,value in q.items():
        temp = value.lower().replace('(','S').replace(')','S').strip()
        multi = 0
        if re.findall(r'^or+\/+\d+-+\d+\s+(and+|not+)',temp):
            multi = 1
            opemul = re.findall(r'^or+\/+\d+-+\d+\s+(and+|not+)',temp)

        if multi == 1:
           parts = re.split(r' not | and ', temp) 
           q_ = ''
           for part in parts:
               if re.findall(r'^or+\/+\d+-+\d+',part) :
                    oper = part.split('/')[0]
                    start = int((part.split('/')[1]).split('-')[0])
                    end = int(part.split('-')[1])
                    
                    part_q = ''           
                    while start <= end:
                        if start == end :
                            part_q = part_q + ' ' + str(start)
                        else:
                            part_q = part_q + ' ' + str(start) + ' ' + oper
                        start += 1
           
               elif re.findall(r'^(or|and)+\/+\d+,+\d+',part):
                     oper = part.split('/')[0]
                     part_q = part.split('/')[1].replace(',',' ' + oper + ' ')
                                    
               if q_ == '':
                   q_ = part_q  + ' ' + opemul[0] + ' ' 
               else:
                   q_ = q_ + part_q 

           q[key] = '( '"S" + q_ + "S" ' )'
                
        
        elif re.findall(r'^or+\/+\d+(-|‐)+\d+',temp):
            oper = temp.split('/')[0]
            start = int((temp.split('/')[1]).split('-')[0])
            end = int(temp.split('-')[1])
            
            temp_q = ''           
            while start <= end:
                if start == end :
                    temp_q = temp_q + ' ' + str(start)
                else:
                    temp_q = temp_q + ' ' + str(start) + ' ' + oper
                start += 1
            
            q[key] = '( ' + temp_q + ' )'
        
        
        elif re.findall(r'^(or|and)+\/+\d+,+\d+',temp):
             oper = temp.split('/')[0]
             temp_q = temp.split('/')[1].replace(',',' ' + oper + ' ')
             q[key] = '( ' + temp_q + ' )'
            
    operations = ['AND','OR','NOT']
    
    for key,value in q.items(): 
        terms = value.replace('(','').replace(')','').split()
        temp = value.replace('(','').replace(')','')
        
        # replace each number with the equivalent line
        for j,term in enumerate(terms):
            if terms[j].isdigit() and ( terms[j-1].lower() in operations or terms[j+1].lower() in operations):
                index = terms[j] 
                temp = temp.replace(str(index),q[int(index)])
        temp = temp.replace('1(','(')
        
        
        q[key] = '( ' + temp + ' )'

    
    return q 




       
    


#******************************************************************************


#******************************************************************************
#******************************************************************************

def main():
    file = sys.argv[1]
    query = open(file,'r').read()
    q_OVID = query.split('\n')
    q_con = Convert_OVID_To_DIALOG(q_OVID)
    q_Dialog =convert_to_dialog('\n'.join(q_con))
    output = open('query.txt','w+')    
    if 'limit' in list(q_Dialog.values())[-1]:
        print(list(q_Dialog.values())[-2])
        output.write(list(q_Dialog.values())[-2])
    else:
        print(list(q_Dialog.values())[-1])
        output.write(list(q_Dialog.values())[-1])
    output.close()
    
#******************************************************************************
#******************************************************************************
    
if __name__ == '__main__':
    main()