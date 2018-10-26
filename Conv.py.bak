#!/usr/bin/env python

import argparse
from File import File

def main():
	parser = argparse.ArgumentParser()
	parser.add_argument("input", type = str, help = "input file")
	parser.add_argument("ext1", type = str, help = "extension of input file")
	parser.add_argument("output", type = str, help = "output file")
	parser.add_argument("ext2", type = str, help = "extension of output file")
	parser.add_argument("-w", dest = "white_list", help = "set file witch keys needs to convert")
	parser.add_argument("-c", dest = "config_file", help = "file witch vaule needs to change")
	parser.add_argument("-v", dest = "det", action = "store_true", help = "high detalisation of procces")
	parser.add_argument("-vv", dest = "hdet", action = "store_true", help = "very high detalisation of procces")
	
	args = parser.parse_args()
	
	f = File(args.input, args.ext1)
	
	flag = str()
	if args.det == True:
		flag = "-v"
	elif args.hdet == True:
		flag = "-vv"

	f.make_dictionary(flag)

	if args.white_list != None:
		f.define_white_list(args.white_list, flag)
	if args.config_file != None:
		f.define_config_file(args.config_file, flag)
		
	f.convert_file_to(args.output, args.ext2, flag)
	
	if flag == "-v" or flag == "-vv":
		print "All done"
		
if __name__ == '__main__':
	status = main()