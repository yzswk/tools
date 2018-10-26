import json
import ConfigParser
import collections
import sqlite3 as sqlite
import copy
from collections import OrderedDict
import sys
import itertools
import os

class File:
    __filename = ''
    __extention = ''
    __list_tuple = list()
    __list_dictionary = list()
    __list_sections = list()

    def __init__(self, f_name, f_ext = ''):
        self.__filename = f_name
        self.__extention = f_ext

    def make_dictionary(self, flag = ''):
        if flag == '-v':
            print "Reading data..."
        elif flag == '-vv':
            print "Input file:", self.__filename
            print "Input extension:", self.__extention
            print "Reading input file..."
            file_size = os.path.getsize(self.__filename)
         
        if self.__extention == "ini":
            conf = ConfigParser.ConfigParser()
            try:
                with open(self.__filename) as data_file:
                    conf.readfp(data_file)
            except:
                print "No such file: ", self.__filename
                sys.exit()
            sections = conf.sections()
            i = 0
            records = 0
            for sect in sections:
                self.__list_tuple.append(conf.items(sect))
                dict_elem = collections.OrderedDict((x, y) for x, y in self.__list_tuple[i])
                records = records + len(dict_elem)
                self.__list_dictionary.append(dict_elem)
                self.__list_sections.append(sect)
                i = i + 1
            
            if flag == '-vv':    
                print "Done, read", file_size, "bytes,", records, "records"
                
        elif self.__extention == "json":
            try:
                with open(self.__filename) as data_file:
                    df_size = sys.getsizeof(data_file)
                    data = json.load(data_file, object_pairs_hook = OrderedDict)
            except:
                print "No such file: ", self.__filename
                sys.exit()                
            data_dict = collections.OrderedDict()
            i = 0
            records = 0
            for each in data:
                self.__list_sections.append(each)
                data_dict = collections.OrderedDict((x, y) for x, y in data[each].iteritems())
                records = records + len(data_dict)
                self.__list_dictionary.append(copy.copy(data_dict))
                if type(self.__list_dictionary[i]) == None:
                    print "Can't conver json to ini or db"
                    sys.exit()
                i = i + 1
            if flag == '-vv':    
                print "Done, read", file_size, "bytes,", records, "records"

        elif self.__extention == "db":
            con = sqlite.connect(self.__filename)
            cur = con.cursor()
            for sect in cur.execute('SELECT name FROM sqlite_master WHERE type = "table"'):
                sect = str(sect).split("'")
                sect = sect[1] 
                self.__list_sections.append(sect)
            records = 0
            for sect in self.__list_sections:
                data_dict = collections.OrderedDict()
                for row in cur.execute('SELECT * FROM {tb}'.\
                                       format(tb=sect)):
                    data_dict[row[0]] = row[1]
                records = records + len(data_dict)
                self.__list_dictionary.append(data_dict)
            con.close()
            
            if flag == '-vv':    
                print "Done, read", file_size, "bytes,", records, "records"            
                
        else:
            print "There is no such file type:", self.__extention
            sys.exit()
            
    def define_white_list(self, white_list = '', flag = ''):
        if flag == '-v' or flag == '-vv':
            print "Reading", white_list,"data..."
        
        if white_list != '':
            list_str = list()
            try:
                f = open(white_list, 'r')
            except:
                print "No such white list: ", white_list
                sys.exit()
            string = f.read()
            f.close()
            list_str = string.splitlines()
            i = 0
            for each in self.__list_dictionary:
                keys = set(list_str).intersection(set(self.__list_dictionary[i].keys()))
                result = {k:self.__list_dictionary[i][k] for k in keys}
                self.__list_dictionary[i] = collections.OrderedDict(result)
                i = i + 1    
    
    def define_config_file(self, config_file = '', flag = ''):
        if flag == '-v' or flag == '-vv':
            print "Reading", config_file, "data..."
        
        if config_file != '':
            try:
                f = open(config_file, 'r')
            except:
                print "No such config file: ", config_file
                sys.exit()
            string = f.read()
            f.close()
            i = 0
            list_str = string.splitlines()
            for each in list_str:
                list_str[i] = each.split(' =')
                list_str[i] = collections.OrderedDict(itertools.izip_longest(*[iter(list_str[i])] * 2))
                i = i + 1
            i = 0
            for each in self.__list_dictionary:
                j = 0
                for ea in list_str:
                    for d1 in each.keys():
                        for d2 in ea.keys():
                            if d1 == d2:
                                self.__list_dictionary[i].update(list_str[j])
                    j = j + 1
                i = i + 1

    def convert_file_to(self, f_name = '', f_ext = '', flag = ''):
        if flag == '-v' or flag == '-vv':
            print "Converting data..."
        if flag == '-vv':
            print "Output file: ", f_name
            print "Output extension: ", f_ext
        
        records = 0
        
        if f_ext == "db":
            con = sqlite.connect(f_name)
            cur = con.cursor()
            i = 0
            for sect in self.__list_sections:
                oldsect = copy.copy(sect)
                sect = sect.replace(" ", "_")
                sect = sect.replace(".", "_")
                if sect != oldsect:
                    print "Warning!!! Section '", oldsect, "' where renamed to '", sect ,"' for correct converting"
                try:
                    cur.execute('CREATE TABLE {tb} (Key TEXT, Value TEXT)'.\
                                format(tb=sect))
                except:
                    print "Error creating table", sect, "already exist"
                    sys.exit()
                records = records + len(self.__list_dictionary[i])
                ttuple = tuple((x,y) for x, y in self.__list_dictionary[i].iteritems())
                cur.executemany('INSERT INTO {tb} VALUES (?, ?)'.\
                                format(tb = sect), ttuple)
                i = i + 1
                con.commit()
            if flag == '-v' or flag == '-vv':
                print "Writing output file..."
            con.close()
            
        elif f_ext == "json":
            i = 0
            dict_copy = collections.OrderedDict()
            for sect in self.__list_sections:
                dict_copy[sect] = collections.OrderedDict((x, y) for x, y in self.__list_dictionary[i].iteritems())
                records = records + len(self.__list_dictionary[i])
                i = i + 1
            self.__list_dictionary = dict_copy
            diction_str = json.dumps(self.__list_dictionary, indent = 4)
            f = open (f_name, 'w')
            if flag == '-v' or flag == '-vv':
                print "Writing output file..."
            f.write(diction_str);
            f.close()
            
        elif f_ext == "ini":
            conf = ConfigParser.ConfigParser()
            i = 0
            for sec in self.__list_sections:
                conf.add_section(sec)
                records = records + len(self.__list_dictionary[i])
                for each in self.__list_dictionary[i]:
                    conf.set(sec, each, self.__list_dictionary[i][each])
                i = i + 1
            if flag == '-v' or flag == '-vv':
                print "Writing output file..."            
            f = open(f_name, 'wb')
            conf.write(f)
            f.close()
        
        else:
            print "No such file type:", f_ext
            sys.exit()
        
        if flag == "-vv":
            print "Done, output file size: ", os.path.getsize(f_name), "records: ", records
