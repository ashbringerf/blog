the JPG and masks are stored with subIFDs;
the RAW is stored as 12bit and with a 12to16 LUT;

@python
import os
from tqdm import tqdm

def jpg_from_dng(src, dst):
	with open(src, 'rb') as fid:
		data = fid.read()
	counter, bool_s, bool_e = 0, 0, 0
	for i in tqdm(range(len(str(data)))):
		# JPG header start hex
		if data[i:i+4] == b'\xff\xd8\xff\xe0':
			index_s, bool_s = i, 1
		# JPG header end hex	
		if data[i:i+2] == b'\xff\xd9':
			index_e, bool_e =i, 1
		if bool_s == 1 and bool_e == 1:
			bool_s, bool_e = 0, 0
			with open(os.path.join(dst, 'dump_' + str(counter) + '.jpg'), 'wb') as fout:
				fout.write(data[index_s: index_e + 2])
			counter += 1
			
jpg_from_dng(r"  ", r"    ")
	