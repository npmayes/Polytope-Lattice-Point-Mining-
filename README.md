# Polytope-Lattice-Point-Mining
import multiprocessing

import time

start = time.perf_counter()

a_0 = 8

# The level 19 system of inequalities
P = Polyhedron(ieqs = [[361*a_0, 506, 314, 146, 2, -118, -214, -286, -334, -358],
                           [361*a_0, 314, 2, -214, -334, -358, -286, -118, 146, 506],
                           [361*a_0, 146, -214, -358, -286, 2, 506, 314, -118, -334],
                           [361*a_0, 2, -334, -286, 146, 506, -118, -358, -214, 314],
                           [361*a_0, -118, -358, 2, 506, -214, -334, 146, 314, -286],
                           [361*a_0, -214, -286, 506, -118, -334, 314, 2, -358, 146],
                           [361*a_0, -286, -118, 314, -358, 146, 2, -334, 506, -214],
                           [361*a_0, -334, 146, -118, -214, 314, -358, 506, -286, 2],
                           [361*a_0, -358, 506, -334, 314, -286, 146, -214, 2, -118],
                           [a_0, 2, 2, 2, 2, 2, 2, 2, 2, 2]])

# The bounding box of the polytope is retrieved and converted to a list of list of
# the relevant integer values
bounding_box = list(P.bounding_box())
bounding_box = [list(i) for i in bounding_box]
bounding_box = [[int(j) for j in i] for i in bounding_box]

# This is the bounding box min, max and range
bounding_box_min = bounding_box[0][0]
bounding_box_max = bounding_box[1][0]
bounding_box_range = [i for i in range(bounding_box_min, bounding_box_max + 1)]

print(bounding_box_range)

def flatten_lists(the_lists):
    flatten = []
    for _list in the_lists:
        flatten += _list
    return flatten

def lattice_point_constructor(_range, send_end):
    result = []
    #count = 0
    for i in bounding_box_range:
        #count += 1
        for k in bounding_box_range:
            for l in bounding_box_range:
                for m in bounding_box_range:
                    for n in bounding_box_range:
                        for r in bounding_box_range:
                            for s in bounding_box_range:
                                for t in bounding_box_range:

                                    x = (i, _range, k, l, m, n, r, s, t)

                                    if ((-9*x[0] - 17*x[1] - 24*x[2] - 30*x[3] - 35*x[4] - 39*x[5] - 42*x[6] - 44*x[7] - 45*x[8]) % 19 == 0
                                        and (a_0 + 2*x[0] + 2*x[1] + 2*x[2] + 2*x[3] + 2*x[4] + 2*x[5] + 2*x[6] + 2*x[7] + 2*x[8]) % 24 == 0
                                        and P.contains(x)):
                                        result.append(x)
    #perc_complete = (count/46.0)*100
    #perc_complete = str(round(perc_complete, 2))
    #print('Program is ' f'{perc_complete}' '% ' 'complete!')
    return send_end.send(result)



processes = []
pipe_list = []

for i in bounding_box_range:
    recv_end, send_end = multiprocessing.Pipe(False)
    p = multiprocessing.Process(target=lattice_point_constructor, args=[i, send_end])
    processes.append(p)
    pipe_list.append(recv_end)
    p.start()

result_list = [x.recv() for x in pipe_list]

for process in processes:
    process.join()

congruent_lattice_points = flatten_lists(result_list)
print(f'\nCongruent Lattice Points: \n\n {congruent_lattice_points}')

with open('lvl_19_a0_6_range_12_interval_12.txt', 'w') as f:
    f.write(str(congruent_lattice_points))

finish = time.perf_counter()
print(f'\nFinished in {round(finish-start, 2)} second(s)\n')
print(f'CPU core count: {multiprocessing.cpu_count()}')
